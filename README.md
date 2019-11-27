### Paging C#
```csharp
// hàm set data cho paging data (đã tạo rồi)
private async IQueryable<T> SetPagingData<T>(PagingData pagingData, IQueryable<T> query)
{
    pagingData.ItemTotal = await query.CountAsync();
    pagingData.PageTotal = pagingData.GetPageTotal();
    pagingData.PageIndex = pagingData.GetPageIndex();
    if (pagingData.PageSize.HasValue)
    {
        var skip = pagingData.GetSkipNumber();
        var take = pagingData.PageSize.Value;
        query = query.Skip(skip).Take(take);
    }
}

public async Task<Acknowledgement<DataWithPaging<Product>>> GetProductListTest(SearchModel searchModel)
{
    var ack = new Acknowledgement<DataWithPaging<Product>>();
    var db = POSReadOnlyContext;
    // Tạo queryable bình thường
    var query = db.Products.Include(i => i.ProductAmountThreshold)
                           .Include(i => i.ProductUnit)
                           .Include(i => i.ProductBatches.Select(j => j.Transactions))
                           .AsQueryable();
    // filter theo search value
    var condition = $"Name.Contains(\"{searchModel.SearchValue}\")";
    query = System.Linq.Dynamic.DynamicQueryable.Where(query, condition);    

    // skip take paging
    query = await SetPagingData(searchModel.PagingData, query);

    // lay data ve memory
    var data = await query.ToListAsync();
    ack.Data = new DataWithPaging<Product>()
    {
        Data = data,
        PagingData = searchModel.PagingData,
    };
    ack.IsSuccess = true;
    return ack;
}
    
```


### Paging javascript

 ```jsx
import {BasePage} from 'BaseComponent/BasePage';
import Pagination from 'components/Pagination';
import I3Table from 'component/I3Table';
import React, {Fragment} from 'react';
export default class TestPage extends BasePage {
    constructor(props) {
        super(props);
        this.state = {
            data: [],
            searchModel: {
                searchValue: "",
                pagingData: {
                    pageIndex: 1,
                    pageSize: 10,
                    pageTotal: 0,
                    itemTotal: 0,
                },
            },
        }
    }
    _getData = () => {
        const { searchModel, } = this.state;
        ajax.post({
            url: `/api/xxx/xxx/`,
            data: JSON.stringify(searchModel),
            successCallback: ack => {
                const _data = ack.data.data // data đã paging
                const _pagingData = ack.data.pagingData // paging data đã được tính toán
                this.clearListAndPushNewItems(this.state.data, _data, () => {
                    this.updateObject(this.state.searchModel.pagingData, _pagingData);
                })
            }
        })
    }
    _gotoNext = () => {
        const { pagingData } = this.state.searchModel;
        if (pagingData.pageIndex < pagingData.pageTotal) {
            this._gotoPage(pagingData.pageIndex + 1);
        }
    }
    _gotoPrev = () => {
        const { pagingData } = this.state.searchModel;
        if (pagingData.pageIndex > 1) {
            this._gotoPage(pagingData.pageIndex - 1);
        }
    }
    _gotoPage = (index) => {
        this.updateObject(this.state.searchModel.pagingData, { pageIndex: index }, this._getData);
    }
    _gotoFirstPage = () => {
        this._gotoPage(1);
    }
    _gotoLastPage = () => {
        this._gotoPage(this.state.searchModel.pagingData.pageTotal);
    }

    childrenRender() {
        const { searchModel, data } = this.state;
        const { pagingData } = searchModel;
        return (
            <div>
                <I3Table 
                    data={data} // list of objects
                    grid={[20, 40, 40]} // tỉ lệ phần trăm độ rộng các cột (sum = 100)
                    headRow={(MuiCell, index) => {
                        return(
                            <Fragment>
                                <MuiCell align="center">
                                    STT
                                </MuiCell>
                                <MuiCell align="left">
                                    Name
                                </MuiCell>
                                <MuiCell>
                                    Age
                                </MuiCell>
                            </Fragment>
                        )   
                    }}
                    bodyRow={(row, MuiCell, index)=>{ // row là một object trong list data đưa vào bên trên
                        return(
                             <Fragment>
                                <MuiCell align="center">
                                    {row.stt}
                                </MuiCell>
                                <MuiCell align="left">
                                    {row.name}
                                </MuiCell>
                                <MuiCell>
                                    {row.age}
                                </MuiCell>
                            </Fragment>
                        )
                    }}
                />
                <Pagination
                    total={pagingData.pageTotal}
                    current={pagingData.pageIndex}
                    onClickNumber={this._gotoPage}
                    onClickNext={this._gotoNext}
                    onClickPrev={this._gotoPrev}
                    onClickFirst={this._gotoFirstPage}
                    onClickLast={this._gotoLastPage}
                />
            </div>
        )
    }
}

```
