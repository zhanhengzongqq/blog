
> 问题表现：点击导出无响应

> 问题排查1：由于第一版本多表连接导致表表呈现速度过慢，程序员将查询方式改为 1+PageSize*7(七个字段分别对应表查询)
> 问题排查2：分页查询只有10条的情况下，查询次数不多（1+10*7），所以没有表现，当导出的时候线上有10万数据(1+100000*7),就炸了

> 在使用先查主表再多次查询关联表的方案来替代多表连接的时候，再查下数据量小的时候性能优化效果明显，但是在查大数据量的时候就崩了，随意要谨慎使用

> 代码节选
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