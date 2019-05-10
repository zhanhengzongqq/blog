- [第一次问题排查](#%E7%AC%AC%E4%B8%80%E6%AC%A1%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5)
  - [问题表现：](#%E9%97%AE%E9%A2%98%E8%A1%A8%E7%8E%B0)
  - [问题排查](#%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5)
  - [问题处理](#%E9%97%AE%E9%A2%98%E5%A4%84%E7%90%86)
  - [问题总结](#%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93)
  - [代码节选](#%E4%BB%A3%E7%A0%81%E8%8A%82%E9%80%89)
- [第二次SQL排查](#%E7%AC%AC%E4%BA%8C%E6%AC%A1sql%E6%8E%92%E6%9F%A5)
  - [改成成原生SQL查询](#%E6%94%B9%E6%88%90%E6%88%90%E5%8E%9F%E7%94%9Fsql%E6%9F%A5%E8%AF%A2)
  - [改造后的SQL](#%E6%94%B9%E9%80%A0%E5%90%8E%E7%9A%84sql)
  - [再次改造后的SQL](#%E5%86%8D%E6%AC%A1%E6%94%B9%E9%80%A0%E5%90%8E%E7%9A%84sql)
  - [相关图片](#%E7%9B%B8%E5%85%B3%E5%9B%BE%E7%89%87)
  - [相关执行计划](#%E7%9B%B8%E5%85%B3%E6%89%A7%E8%A1%8C%E8%AE%A1%E5%88%92)
### 第一次问题排查
#### 问题表现：
> 点击导出无响应

#### 问题排查
1. 由于第一版本多表连接导致表表呈现速度过慢，程序员将查询方式改为 1+PageSize*7(七个字段分别对应表查询)
2. 分页查询只有10条的情况下，查询次数不多（1+10*7），所以没有表现，当导出的时候线上有10万数据(1+100000*7),就炸了

#### 问题处理
1. 改为多表连接，建立合适索引
2. 限制导出条数如100条
   
#### 问题总结
> 在使用先查主表再多次查询关联表的方案来替代多表连接的时候，再查下数据量小的时候性能优化效果明显，但是在查大数据量的时候就崩了，随意要谨慎使用

#### 代码节选
```csharp
var list = await DisabledInfoQueryEntry.GetDisabledInfoListAsync(disabledInfoPageQueryFilter);
return JsonPagedResult(await list.AsPagedAsync(async m => new DisabilityInfoModel()
{
    ...
    AccountNature = (await DisabilityScaleInfoQueryEntry.GetDisabilityScaleInfoByIdNumberAsync(m.IdNumber))?.R3?.GetDescription() ?? "暂无数据",
    PovertyFileEstablishment = (await DisabilityScaleInfoQueryEntry.GetDisabilityScaleInfoByIdNumberAsync(m.IdNumber))?.R10.TextToEnumList(typeof(PovertyFileEstablishment)).Join(',') ?? "暂无数据",
    LowerProtectionismState = (await DisabilityScaleInfoQueryEntry.GetDisabilityScaleInfoByIdNumberAsync(m.IdNumber))?.R8?.GetDescription() ?? "暂无数据",
    Insurance = "",
    StudyStatus = (await DisabledInfoQueryEntry.GetProvincialInterfaceSchoolInfoByIdNumberAsync(m.IdNumber))?.StudyStatus?.GetDescription() ?? "暂无数据",
    IsEmployment = (await DisabilityScaleInfoQueryEntry.GetDisabilityScaleInfoByIdNumberAsync(m.IdNumber))?.R16?.GetDescription() ?? "暂无数据",
    IsWorkToBe = await CvQueryEntry.IsExistCvRecommendAsync(m.IdNumber) ? "有" : "无",
    IsActivity = await ActivityRegistrationInfoQueryEntry.IsExistCvActivityAsync(m.IdNumber) ? "有" : "无"
}));

```            
### 第二次SQL排查

#### 改成成原生SQL查询
> 其中TwoSubsid为10w级别表，下面的速度还是非常慢，其实还是犯了跟上面相同的错误，这个模式的SQL语句还是多次查询的结果，只不过是SQL级别的
```sql
select di.Id, convert(int,(left(di.AreaCode,6))) as Postal,di.AreaCode,di.Name,di.Gender,di.Grade,di.DateBirth,di.DisabilityNo,di.CreateTime,
di.DisabilityType,di.DisabilityLevel,di.Guardian,di.Mobile,di.IdNumber,di.Marriage,
dsi.R8 as LowerProtectionismState,dsi.R3 as AccountNature,dsi.r10 as PovertyFileEstablishment, dsi.R16 as IsEmployment,
(select isnull((select top(1) 1 from TwoSubsidy as dsr where dsr.IdNumber=di.IdNumber and dsr.SubsidyType=4),0)) as LifebutieState,
(select isnull((select top(1) 1 from TwoSubsidy as dsr where dsr.IdNumber=di.IdNumber and dsr.SubsidyType=8),0)) as NursebutieState,
(select isnull((select top(1) 1 from CV where CV.IdentityNo=di.IdNumber and Cv.IsDeleted=0),0)) as IsWorkToBe,
(select isnull((select top(1) 1 from ActivityRegistrationInfo as ari where ari.IdNumber=di.IdNumber and ari.IsDeleted=0),0)) as IsActivity 
from DisabledInfo as di 
left join DisabilityScaleInfo as dsi on di.IdNumber=dsi.R42  
where (di.IsDeleted=0 or di.IsDeleted is null)  and (dsi.IsDeleted=0 or dsi.IsDeleted is null)
```

#### 改造后的SQL
> 速度明显提升了
```sql
select di.Id, convert(int,(left(di.AreaCode,6))) as Postal,di.AreaCode,di.Name,di.Gender,di.Grade,di.DateBirth,di.DisabilityNo,di.CreateTime,
di.DisabilityType,di.DisabilityLevel,di.Guardian,di.Mobile,di.IdNumber,di.Marriage,
dsi.R8 as LowerProtectionismState,dsi.R3 as AccountNature,dsi.r10 as PovertyFileEstablishment, dsi.R16 as IsEmployment,
ISNULL(ts1.SubsidyType, 0) AS LifebutieState,
ISNULL(ts2.SubsidyType, 0) AS NursebutieState,
(select isnull((select top(1) 1 from CV where CV.IdentityNo=di.IdNumber and Cv.IsDeleted=0),0)) as IsWorkToBe,
(select isnull((select top(1) 1 from ActivityRegistrationInfo as ari where ari.IdNumber=di.IdNumber and ari.IsDeleted=0),0)) as IsActivity 
from DisabledInfo as di 
left join DisabilityScaleInfo as dsi on di.IdNumber=dsi.R42  
LEFT JOIN dbo.TwoSubsidy AS ts1 ON di.IdNumber = ts1.IdNumber AND ts1.SubsidyType = 4
LEFT JOIN dbo.TwoSubsidy AS ts2 ON di.IdNumber = ts2.IdNumber AND ts2.SubsidyType = 8
where (di.IsDeleted=0 or di.IsDeleted is null)  and (dsi.IsDeleted=0 or dsi.IsDeleted is null)

```

#### 再次改造后的SQL
> 同时再 *DisabilityScaleInfo* 和 *TwoSubsidy* 这两张大表的IdNumber上建了聚集索引

``` sql
select di.Id, convert(int,(left(di.AreaCode,6))) as Postal,di.AreaCode,di.Name,di.Gender,di.Grade,di.DateBirth,di.DisabilityNo,di.CreateTime,
di.DisabilityType,di.DisabilityLevel,di.Guardian,di.Mobile,di.IdNumber,di.Marriage,
dsi.R8 as LowerProtectionismState,dsi.R3 as AccountNature,dsi.r10 as PovertyFileEstablishment, dsi.R16 as IsEmployment,
ISNULL(ts1.SubsidyType, 0) AS LifebutieState,
ISNULL(ts2.SubsidyType, 0) AS NursebutieState,
ISNULL(cv.IdentityNo, 0) AS IsWorkToBe,
ISNULL(ar.IdNumber, 0) AS IsActivity
from DisabledInfo as di 
left join DisabilityScaleInfo as dsi on di.IdNumber=dsi.R42  
LEFT JOIN dbo.TwoSubsidy AS ts1 ON di.IdNumber = ts1.IdNumber AND ts1.SubsidyType = 4
LEFT JOIN dbo.TwoSubsidy AS ts2 ON di.IdNumber = ts2.IdNumber AND ts2.SubsidyType = 8
LEFT JOIN dbo.Cv AS cv ON di.IdNumber = cv.IdentityNo
LEFT JOIN (SELECT DISTINCT(IdNumber) FROM dbo.ActivityRegistrationInfo)ar ON di.IdNumber = ar.IdNumber
where (di.IsDeleted=0 or di.IsDeleted is null)  and (dsi.IsDeleted=0 or dsi.IsDeleted is null)
```

#### 相关图片
![](/images/0012.png?raw=true)

#### 相关执行计划
- 显示更详细的执行计划：SET STATISTICS PROFILE ON;
![](/images/0013.png?raw=true)
![](/images/0014.png?raw=true)
![](/images/0015.png?raw=true)
  

  