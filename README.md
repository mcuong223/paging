### Paging C#
```csharp
    // hàm set data cho paging data (đã tạo rồi)
    private async Task SetPagingData<T>(PagingData pagingData, IQueryable<T> query)
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
            await SetPagingData(searchModel.PagingData, query);
            
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
    import React from 'react';
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
                    <RenderData
                        xxx={data}
                    />
                    {/* render data các thứ*/}
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
