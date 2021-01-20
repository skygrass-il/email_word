<template>
  <div>
    <div class="text-right mt-4">
      <button class="btn btn-primary" @click="openCouponModal(true)">建立新的優惠劵</button>
    </div>
    <table class="table mt-4">
      <thead>
        <tr>
          <th>名稱</th>
          <th width="180">折扣百分比</th>
          <th width="150">到期日</th>
          <th width="150">是否啟用</th>
          <th width="80">編輯</th>
          <th width="80">刪除</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item,key) in coupons" :key="key">
          <td>{{ item.title }}</td>
          <td>{{ item.percent }}%</td>
          <td>{{ item.due_date | time}}</td>
          <td>
              <span v-if="item.is_enabled" class="text-success">啟用</span>
              <span v-else class="text-muted">未啟用</span>
          </td>
          <td>
              <button class="btn btn-outline-primary btn-sm" @click="openCouponModal(false,item)">編輯</button>
          </td>
          <td>
              <button class="btn btn-outline-danger btn-sm" @click="openCouDelModal(item)">刪除</button>
          </td>
        </tr>
      </tbody>
    </table>
    <!--Modal-->
    <div class="modal fade" id="couponModal" tabindex="-1" role="dialog"
      aria-labelledby="exampleModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">新增優惠劵</h5>
          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        <div class="form-group">
              <label for="title">標題</label>
              <input type="text" class="form-control" id="title" v-model="tempCoupon.title"
                placeholder="請輸入標題">
        </div>
        <div class="form-group">
              <label for="coupon_code">優惠碼</label>
              <input type="text" class="form-control" id="coupon_code" v-model="tempCoupon.code"
                placeholder="請輸入優惠碼">
        </div>
        <div class="form-group">
              <label for="due_date">到期日</label>
              <input type="date" class="form-control" id="due_date" v-model="due_date"
                placeholder="請輸入到期日">
        </div>
        <div class="form-group">
              <label for="price">折扣百分比</label>
              <input type="number" class="form-control" id="price" v-model="tempCoupon.percent"
                placeholder="請輸入折扣百分比">
        </div>
        <div class="form-group">
              <div class="form-check">
                <input class="form-check-input" type="checkbox" v-model="tempCoupon.is_enabled" :true-value="1" :false-value="0"
                  id="is_enabled">
                <label class="form-check-label" for="is_enabled">
                  是否啟用
                </label>
              </div>
        </div>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">取消</button>
        <button type="button" class="btn btn-primary" @click="updateCoupon">更新優惠劵</button>
      </div>
    </div>
  </div>
</div>

<div class="modal" id="delCouponModal" tabindex="-1">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">刪除優惠劵</h5>
        <button type="button" class="btn-close" data-dismiss="modal" aria-label="Close"></button>
      </div>
      <div class="modal-body">
        是否刪除 <strong class="text-danger">{{ tempCoupon.title }}</strong> 優惠劵(刪除後將無法恢復)。
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">取消</button>
        <button type="button" class="btn btn-danger" @click="delCoupon">確認刪除</button>
      </div>
    </div>
  </div>
</div>

  </div>
</template>

<script>
export default {
  data() {
    return {
      coupons: [],
      tempCoupon:{
          title:'',
          is_enabled:0,
          percent:100,
          due_date:0,
          cade:'',
      },
      due_date:new Date(),
      isNew:false,
    };
  },
  watch:{
      due_date() {  
          const vm = this;
          const timestamp = Math.floor(new Date(vm.due_date)/1000);
          vm.tempCoupon.due_date = timestamp;
      }
  },
  methods: {
    getCoupons(page = 1) {
      const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/coupons?page=${page}`;
      const vm = this;
      this.$http.get(api).then((response) => {
        console.log(response.data);
        vm.coupons = response.data.coupons;
      });
    },
    openCouponModal(isNew,item) {
        const vm  = this;
        vm.isNew = isNew;
        if(isNew){
            vm.due_date = new Date();
            this.tempCoupon = {};
        }else{
            // this.tempCoupon = Object.assign({},item);
            this.tempCoupon = {...item};
            // if(typeof(vm.tempCoupon.due_date) !== Number) {           //因為原本用日期沒轉timestamp才需要這段if判斷
            //    const timestamp = Math.floor(new Date(vm.due_date)/1000);
            //    vm.tempCoupon.due_date = timestamp;
            // }
            const dateAndTime = new Date(vm.tempCoupon.due_date*1000).toISOString().split('T'); //編輯Modal裡的顯示日期格式
            vm.due_date = dateAndTime[0];
        }
        $('#couponModal').modal('show');
    },
    openCouDelModal(item) {
        const vm = this;
        vm.tempCoupon = item;
        $('#delCouponModal').modal('show');
    },
    updateCoupon() {
        const vm = this;
        if(vm.isNew) {
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/coupon`;
            this.$http.post(api,{data:vm.tempCoupon}).then((response)=>{
                console.log(response,vm.tempCoupon);
                 if(response.data.success){
                $('#couponModal').modal('hide');
                vm.getCoupons();
            }else{
                $('#couponModal').modal('hide');
                vm.getCoupons();
                console.log('新增失敗');
            }
            });
        }else {
            const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/coupon/${vm.tempCoupon.id}`;
           vm.due_date = new Date(vm.tempCoupon.due_date*1000);  //一修改編輯Modal日期watch due_date將其轉為timestamp至vm.tempCoupon.due_date
           this.$http.put(api,{data:vm.tempCoupon}).then((response)=>{
                console.log(response);
                 if(response.data.success){
                $('#couponModal').modal('hide');
                vm.getCoupons();
            }else{
                $('#couponModal').modal('hide');
                vm.getCoupons();
                console.log('修改失敗');
            }
            });
        }
        // let api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/coupon`;
        // let httpMethod = 'post';
        // const vm = this;
        // if(!vm.isNew){
        //     api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/coupon/${vm.tempCoupon.id}`;
        //     httpMethod = 'put';
        // }
        // // console.log(process.env.APIPATH,process.env.CUSTOMPATH);
        // this.$http[httpMethod](api,{data:vm.tempCoupon}).then((response)=>{
        //     console.log(response.data);
        //     if(response.data.success){
        //         $('#couponModal').modal('hide');
        //         vm.getCoupons();
        //     }else{
        //         $('#couponModal').modal('hide');
        //         vm.getCoupons();
        //         console.log('新增失敗');
        //     }
        // });
    },
    delCoupon() {
        const vm  = this;
        const api =`${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/coupon/${vm.tempCoupon.id}`;
        this.$http.delete(api).then((response)=>{
            console.log(response,vm.tempCoupon);
            if(response.data.success){
                $('#delCouponModal').modal('hide');
                vm.getCoupons();
            }
        });
    },
  },
  created() {
      this.getCoupons();
  }
};
</script>







