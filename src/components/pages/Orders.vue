<template>
    <div>
        <loading :active.sync="isLoading"></loading>
        <table class="table mt-4">
            <thead>
                <tr>
                    <th width="120">購買時間</th>
                    <th>Email</th>
                    <th>購買款項</th>
                    <th width="120">應付金額</th>
                    <th width="120">是否付款</th>
                    <th width="80">編輯</th>
                </tr>
            </thead>
            <tbody v-if="orders.length">
                <tr v-for="(item,key) in sortOrder" :key="key" :class="{'text-muted':!item.is_paid}">
                    <td>{{ item.create_at | time}}</td>
                    <td><span v-if="item.user">{{ item.user.email }}</span></td>
                    <td>
                    <ul class="list-unstyled">
                        <li v-for="(product,i) in item.products" :key="i">
                            {{ product.product.title }} 數量:{{ product.qty }}
                            {{ product.product.unit }}
                        </li>
                    </ul>
                    </td>
                    <td class="text-right">{{ item.total | currency}}</td>
                    <td>
                        <span v-if="item.is_paid" class="text-success">已付款</span>
                        <span v-else class="text-muted">尚未啟用</span>
                    </td>
                    <td>
                        <button class="btn btn-outline-primary btn-sm" @click="openOrderModal(item)">編輯</button>
                    </td>
                </tr>
            </tbody>
        </table>

        <Pagination :pages="pagination" @emitPages="getOrders"></Pagination>

        <!--Modal-->
    <div class="modal fade" id="orderModal" tabindex="-1" role="dialog"
      aria-labelledby="exampleModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
        <div class="modal-header">
        <h5 class="modal-title">修改訂單</h5>
          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        <div class="form-group">
              <label for="buy_date">購買時間</label>
              <input type="date" class="form-control" id="buy_date" v-model="tempOrder.create_at"
                placeholder="請輸入購買時間">
        </div>
        <div class="form-group">
              <label for="order_email">Email</label>
              <input type="text" class="form-control" id="order_email" v-model="tempOrder.user.email"
                placeholder="請輸入電子郵件">
        </div>
        <div class="form-group">
              <div class="form-check">
                <input class="form-check-input" type="checkbox" v-model="tempOrder.is_paid"
                  id="is_paid">
                <label class="form-check-label" for="is_paid">
                  是否啟用
                </label>
              </div>
        </div>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">取消</button>
        <button type="button" class="btn btn-primary" @click="updateOrder">更新訂單</button>
      </div>
    </div>
  </div>
</div>

    </div>
</template>

<script>
import Pagination from '../Pagination';
export default {
    data() {
        return {
            orders:[],
            tempOrder:{
                create_at:0,
                is_paid:false,
                user:{
                    email:'',
                },
            },
            isLoading:false,
            pagination:{},
        }
    },
    components:{
        Pagination,
    },
    methods:{
        getOrders(page=1) {
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/orders?page=${page}`;
            const vm = this;
            // console.log(process.env.APIPATH,process.env.CUSTOMPATH);
            vm.isLoading = true;
            this.$http.get(api).then((response)=>{
                console.log(response.data);
                vm.isLoading = false;
                vm.orders = response.data.orders;
                vm.pagination = response.data.pagination;
            });
        },
        openOrderModal(item) {
            this.tempOrder = {...item};
            $('#orderModal').modal('show');
        },
        updateOrder() {
            const vm = this;
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/order/${vm.tempOrder.id}`;
            this.$http.put(api,{data:vm.tempOrder}).then((response)=>{
                console.log(response.data);
                if(response.data.success) {
                    $('orderModal').modal('hide');
                    vm.getOrder();
                }else {
                    $('orderModal').modal('hide');
                    vm.getOrder();
                    console.log('更新失敗');
                }
            });
        },
    },
    computed:{
        sortOrder() {
            const vm = this;
            if(vm.orders.length) {
               return vm.orders.sort((a,b)=>{
                    return b[b.is_paid] - a[a.is_paid];
                });
            }
            
    //          const vm = this;
    //   let newOrder = [];
    //   if (vm.orders.length) {
    //     newOrder = vm.orders.sort((a, b) => {
    //       const aIsPaid = a.is_paid ? 1 : 0;
    //       const bIsPaid = b.is_paid ? 1 : 0;
    //       return bIsPaid - aIsPaid;
    //     });
    //   }
    //   return newOrder;

        }
    },
    created() {
        this.getOrders();
    }
}
</script>





