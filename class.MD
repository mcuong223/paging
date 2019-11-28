```csharp
public class SearchModel
{
    public string SearchValue { get; set; }
    public PagingData PagingData { get; set; }
}
public class AckWithPaging<T> : Acknowledgement<List<T>>
{
    public PagingData PagingData { get; set; }
}
public class PagingData
{
    public int PageIndex { get; set; }
    public int? PageSize { get; set; }
    public int ItemTotal { get; set; }
    public int PageTotal { get; set; }

    public int GetPageTotal()
    {
        if (PageSize.HasValue == false)
        {
            return 1;
        }
        int pageTotal = 0;

        if (ItemTotal >= 0 || PageSize >= 0)
        {
            pageTotal = (int)(ItemTotal / PageSize) + (ItemTotal % PageSize > 0 ? 1 : 0);
        }

        return pageTotal;
    }

    public int GetSkipNumber()
    {
        if (PageSize.HasValue == false)
        {
            return 0;
        }
        else
        {
            if (PageIndex <= 0 || PageTotal <= 0) return 0;                    // Validate
            if (PageTotal < PageIndex) return (PageTotal - 1) * PageSize.Value;    // Sẽ lấy page cuối
            return (PageIndex - 1) * PageSize.Value;
        }
    }

    public int GetPageIndex()
    {
        if (ItemTotal == 0)
        {
            return 0;
        }
        if (PageIndex == 0)
        {
            return 1;
        }
        if (PageIndex > PageTotal)
        {
            return PageTotal;
        }
        return PageIndex;
    }
}

```
