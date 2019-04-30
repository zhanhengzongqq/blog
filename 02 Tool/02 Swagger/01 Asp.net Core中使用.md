
- [使用的组件](#%E4%BD%BF%E7%94%A8%E7%9A%84%E7%BB%84%E4%BB%B6)
- [添加对Enum的支持](#%E6%B7%BB%E5%8A%A0%E5%AF%B9enum%E7%9A%84%E6%94%AF%E6%8C%81)
- [对于多程序集的支持](#%E5%AF%B9%E4%BA%8E%E5%A4%9A%E7%A8%8B%E5%BA%8F%E9%9B%86%E7%9A%84%E6%94%AF%E6%8C%81)
- [对于上传控件的支持](#%E5%AF%B9%E4%BA%8E%E4%B8%8A%E4%BC%A0%E6%8E%A7%E4%BB%B6%E7%9A%84%E6%94%AF%E6%8C%81)
- [其他一些参考资料](#%E5%85%B6%E4%BB%96%E4%B8%80%E4%BA%9B%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99)

### 使用的组件

*第三方组件 Swashbuckle.AspNetCore.Swagger*
>[使用帮助参考官网](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)

### 添加对Enum的支持

> [参考于StackOverFlow](https://stackoverflow.com/questions/36452468/swagger-ui-web-api-documentation-present-enums-as-strings)

1. 设置SwaggerGen初始化选项
```csharp
serviceCollection.AddSwaggerGen(options =>
            {
                ...
                //生成枚举类型注释文档
                options.DocumentFilter<EnumDocumentFilter>();
                .....
            });
```

2. EnumDocumentFilter 源码
```csharp
    /// <summary>
    /// Swagge枚举类型过滤器 生成枚举类型文档
    /// </summary>
    public class EnumDocumentFilter : IDocumentFilter
    {
        public void Apply(SwaggerDocument swaggerDoc, DocumentFilterContext context)
        {
            // add enum descriptions to result models
            foreach (var schemaDictionaryItem in swaggerDoc.Definitions)
            {
                var schema = schemaDictionaryItem.Value;
                foreach (var propertyDictionaryItem in schema.Properties)
                {
                    var property = propertyDictionaryItem.Value;
                    var propertyEnums = property.Enum;
                    if (propertyEnums != null && propertyEnums.Count > 0)
                    {
                        property.Description += DescribeEnum(propertyEnums);
                    }
                }
            }

            if (swaggerDoc.Paths.Count <= 0) return;

            // add enum descriptions to input parameters
            foreach (var pathItem in swaggerDoc.Paths.Values)
            {
                DescribeEnumParameters(pathItem.Parameters);

                // head, patch, options, delete left out
                var possibleParameterisedOperations = new List<Operation> { pathItem.Get, pathItem.Post, pathItem.Put };
                possibleParameterisedOperations.FindAll(x => x != null)
                    .ForEach(x => DescribeEnumParameters(x.Parameters));
            }
        }

        private static void DescribeEnumParameters(IList<IParameter> parameters)
        {
            if (parameters == null) return;

            foreach (var param in parameters)
            {
                if (param.Extensions.ContainsKey("enum") && param.Extensions["enum"] is IList<object> paramEnums &&
                    paramEnums.Count > 0)
                {
                    param.Description += DescribeEnum(paramEnums);
                }
            }
        }

        private static string DescribeEnum(IEnumerable<object> enums)
        {
            var enumDescriptions = new List<string>();
            Type type = null;
            foreach (var enumOption in enums)
            {
                if (type == null) type = enumOption.GetType();
                string enumDescription = null;
                if (enumOption is Enum eumtype)
                {
                    enumDescription = GetEnumDescription((Enum)enumOption);
                }
                //判断是否有描叙说明
                if (!string.IsNullOrWhiteSpace(enumDescription))
                {
                    enumDescriptions.Add($"{Convert.ChangeType(enumOption, type.GetEnumUnderlyingType())} = {Enum.GetName(type, enumOption)}   注释说明：{enumDescription} ");
                }
                else
                {
                    enumDescriptions.Add($"{Convert.ChangeType(enumOption, type.GetEnumUnderlyingType())} = {Enum.GetName(type, enumOption)} ");
                }
            }
            return $"{Environment.NewLine}{string.Join(Environment.NewLine, enumDescriptions)}";
        }

        private static string GetEnumDescription(Enum value)
        {
            FieldInfo fi = value.GetType().GetField(value.ToString());
            DescriptionAttribute[] attributes = (DescriptionAttribute[])fi.GetCustomAttributes(typeof(DescriptionAttribute), false);
            return (attributes != null && attributes.Length > 0) ? attributes[0].Description : null;
        }
    }
```
### 对于多程序集的支持

![图片](images/01.png?raw=true)
1. 选中XML文档文件
2. 设置输出目录到站点程序集，可用相对路径如：..\CCMS.Host\wwwroot\swagger\xmls\CCMS.MainModule.App.xml
3. 设置SwaggerGen初始化选项
```csharp
serviceCollection.AddSwaggerGen(options =>
            {
                ...
                //生成枚举类型注释文档
                options.IncludeXmlComments(GetSwaggerXmlDoc, true);
                ...
            });
```
4. 多XMl合并源码
```csharp
/// <summary>
        /// 获取处理当前解决方案下所有的方法模型的注释xml
        /// </summary>
        /// <returns></returns>
        public static XPathDocument GetSwaggerXmlDoc()
        {
            var directory = Directory.GetCurrentDirectory() + @"\wwwroot\swagger\xmls";
            // 找到所有的注释文档， 根据项目情况加上通配符。
            var files = Directory.GetFiles(directory, "CCMS*.xml").ToList();
            // 找到主项目注释文档
            var mainFile = files.Where(u => u.Contains("Host")).FirstOrDefault();
            if (string.IsNullOrEmpty(mainFile))
                return null;
            files.Remove(mainFile);
            if (files.Count == 0)
                return null;
            var mainDoc = new XmlDocument();
            mainDoc.Load(mainFile);
            var xPathNavigator = mainDoc.CreateNavigator();
            var membersNode = xPathNavigator.SelectSingleNode("//members");
            files.ForEach(file =>
            {
                var otherDoc = new XmlDocument();
                otherDoc.Load(file);
                var otherXPathNavigator = otherDoc.CreateNavigator();
                var nodes = otherXPathNavigator.Select("//member");
                while (nodes.MoveNext())
                {
                    membersNode.AppendChild(nodes.Current);
                }
            });

            using (var stream = new MemoryStream())
            {
                mainDoc.Save(stream);
                stream.Seek(0, SeekOrigin.Begin);
                return new XPathDocument(stream);
            }
        }
```

### 对于上传控件的支持
> [参考于StackOverFlow](https://stackoverflow.com/questions/36892604/asp-net-core-swagger-help-pages-for-iformfile)
1. 设置SwaggerGen初始化选项
```csharp
serviceCollection.AddSwaggerGen(options =>
            {
                ...
                //配置文件上传按钮
                options.OperationFilter<FormFileOperationFilter>();
                ...
            });
```

2. FormFileOperationFilter源码
```csharp
/// <summary>
/// Adds support for IFormFile parameters in Swashbuckle.
/// </summary>
public class FormFileOperationFilter : IOperationFilter
{
    private const string FormDataMimeType = "multipart/form-data";
    private static readonly string[] FormFilePropertyNames =
        typeof(IFormFile).GetTypeInfo().DeclaredProperties.Select(x => x.Name).ToArray();

    public void Apply(Operation operation, OperationFilterContext context)
    {
        if (context.ApiDescription.ParameterDescriptions.Any(x => x.ModelMetadata.ContainerType == typeof(IFormFile)))
        {
            var formFileParameters = operation
                .Parameters
                .OfType<NonBodyParameter>()
                .Where(x => FormFilePropertyNames.Contains(x.Name))
                .ToArray();
            var index = operation.Parameters.IndexOf(formFileParameters.First());
            foreach (var formFileParameter in formFileParameters)
            {
                operation.Parameters.Remove(formFileParameter);
            }

            var formFileParameterName = context
                .ApiDescription
                .ActionDescriptor
                .Parameters
                .Where(x => x.ParameterType == typeof(IFormFile))
                .Select(x => x.Name)
                .First();
            var parameter = new NonBodyParameter()
            {
                Name = formFileParameterName,
                In = "formData",
                Description = "The file to upload.",
                Required = true,
                Type = "file"
            };
            operation.Parameters.Insert(index, parameter);

            if (!operation.Consumes.Contains(FormDataMimeType))
            {
                operation.Consumes.Add(FormDataMimeType);
            }
        }
    }
}

```

### 其他一些参考资料
[Asp.net中使用,以及汉化](http://www.cnblogs.com/yanweidie/p/5709113.html)