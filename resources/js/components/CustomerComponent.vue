<template>
  <div id="customer">

    <div class="row justify-content-center">
      <div class="col-md-12">
        <div class="card">
          <div class="card-header">
            <h4 class="card-title">Customers</h4>
            <div class="card-tools" style="position: absolute;right:1rem;top:.5rem;">
              <button type="button" class="btn btn-info" @click="create">Add New <i class="fas fa-plus"></i></button>
              <button type="button" class="btn btn-primary" @click="reload">Reload <i class="fas fa-sync"></i></button>
            </div>
          </div>

            <div class="card-body">
              <div class="mb-3">
                <div class="row">
                  <div class="col-md-2" style="margin-top: 10px" >
                    <strong>Search By : </strong>
                  </div>
                  <div class="col-md-3">
                    <select v-model="queryField" class="form-control" id="fields">
                      <option value="name">Name</option>
                      <option value="email">Email</option>
                      <option value="phone">Phone</option>
                      <option value="address">Address</option>
                      <option value="amount">Amount</option>
                    </select>
                  </div>
                  <div class="col-md-7">
                    <input type="text" v-model="query" class="form-control" placeholder="Search">
                  </div>
                </div>
              </div>
                <div class="table-responsive">
                    <table class="table table-hover table-bordered table-striped ">
                      <thead>
                        <tr>
                          <th scope="col">#</th>
                          <th scope="col">Name</th>
                          <th scope="col">Email</th>
                          <th scope="col">Phone</th>
                          <th scope="col">Amount</th>
                          <th scope="col" class="text-center">Action</th>
                        </tr>
                      </thead>
                      <tbody>
                          <tr v-show="customers.length" v-for="(customer, index) in customers" :key="customer.id">
                            <th scope="row">{{ index + 1 }}</th>
                            <td>{{ customer.name }}</td>
                            <td>{{ customer.email }}</td>
                            <td>{{ customer.phone }}</td>
                            <td>{{ customer.amount }}</td>
                            <td class="text-center">
                              <button type="button" @click="detail(customer)"  class="btn btn-info btn-sm">
                                <i class="fa fa-eye" aria-hidden="true"></i>
                              </button>
                              <button type="button" @click="edit(customer)" class="btn btn-primary btn-sm">
                                <i class="fa fa-edit" aria-hidden="true"></i>
                              </button>
                              <button type="button" @click="destroy(customer)" class="btn btn-danger btn-sm">
                                <i class="fa fa-trash-alt" aria-hidden="true"></i>
                              </button>
                            </td>  
                          </tr>
                          <tr v-show="!customers.length">
                            <td colspan="6">
                              <div class="alert alert-danger" role="alert">
                                Sorry : No Data Found.

                              </div>
                            </td>
                          </tr>
                      </tbody>
                    </table>  
                    <pagination
                      v-if="pagination.last_page > 1"
                      :pagination="pagination"
                      :offset="5"
                      @paginate="query ==='' ? getData() : searchData()"
                    ></pagination>                                     
                </div>
            </div>
        </div>
      </div>
    </div>
  <div class="modal fade" id="customerModal" tabindex="-1" role="dialog" aria-labelledby="customerModalTitle" aria-hidden="true">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="customerModalTitle">{{ editMode ? "Edit" : "Add New" }} Customer</h5>
          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <form @submit.prevent="editMode ? update() : createStore()" @keydown="form.onKeydown($event)">
          <div class="modal-body">
            <alert-error :form="form"></alert-error>
            <div class="form-group">
              <label>Name</label>
              <input v-model="form.name" type="text" name="name"
                class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
              <has-error :form="form" field="name"></has-error>
            </div>

            <div class="form-group">
              <label>Email</label>
              <input v-model="form.email" type="email" name="email"
                class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
              <has-error :form="form" field="email"></has-error>
            </div>

            <div class="form-group">
              <label>Phone</label>
              <input v-model="form.phone" type="tel" name="phone"
                class="form-control" :class="{ 'is-invalid': form.errors.has('phone') }">
              <has-error :form="form" field="phone"></has-error>
            </div>
            <div class="form-group">
              <label>Address</label>
              <textarea v-model="form.address" name="address"
                class="form-control" :class="{ 'is-invalid': form.errors.has('address') }"></textarea>
              <has-error :form="form" field="address"></has-error>
            </div>
            <div class="form-group">
              <label>Amount</label>
              <input v-model="form.amount" type="text" name="amount"
                class="form-control" :class="{ 'is-invalid': form.errors.has('amount') }">
              <has-error :form="form" field="amount"></has-error>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
            <button :disabled="form.busy" type="submit" class="btn btn-primary">Save Changes</button>
          </div>
        </form>    
      </div>
    </div>
  </div>
  <!--Show Modal -->
  <div class="modal fade" id="detailModal" tabindex="-1" role="dialog" aria-labelledby="detailModalLabel"     aria-hidden="true">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header" style="background:#ECECEC;">
          <h4 class="modal-title" id="detailModalLabel">{{ form.name }}</h4>
          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body" style="background:#F7F7F7;">
          <strong>Email : {{ form.email }}</strong>
          <br>
          <strong>Phone : {{ form.phone }}</strong>
          <br>
          <strong>Amount : {{ form.amount }}</strong>
          <br>
          <strong>Address :</strong>
          <address>{{ form.address }}</address>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
        </div>
      </div>
    </div>
  </div>
    <vue-progress-bar></vue-progress-bar>
    <vue-snotify></vue-snotify>
  </div>
</template>

<script>
    export default {
      data() {
        return{
          editMode: false,
          query: '',
          queryField: 'name',
          customers: [],
          form : new Form({
            id:'',
            name:'',
            email:'',
            phone:'',
            address:'',
            amount:'',

          }),
          pagination: {
            current_page: 1,
          }
        }
      },
      watch: {
        query:function(newQ, old){
          if(newQ ==='')
          {
            this.getData()
          }
          else
          {
            this.searchData()
          }
        }
      },

      mounted() {
            console.log('Component mounted.')
            this.getData();
        },

      methods: {
        getData(){
          this.$Progress.start()
          axios.get('/vuedataviewer/api/customers?page=' + this.pagination.current_page)
            .then(response => {
              this.customers = response.data.data
              this.pagination = response.data.meta
              this.$Progress.finish()
            })
            .catch(e => {
              console.log(e)
              this.$Progress.fail()
            })
        },
        searchData(){
          this.$Progress.start()
          axios.get('/vuedataviewer/api/search/customers/'+this.queryField+'/'+this.query+'?page=' + this.pagination.current_page)
          .then(response => {
              this.customers = response.data.data
              this.pagination = response.data.meta
              this.$Progress.finish()
            })
            .catch(e => {
              console.log(e)
              this.$Progress.fail()
            })
        },
        reload(){
          this.getData()
          this.query = ''
          this.queryField ='name'

        },
        create(){
          this.editMode = false
          this.detailMode = false
          this.form.reset()
          this.form.clear()
          $('#customerModal').modal('show')

        },
        createStore(){
          this.$Progress.start()
          this.form.busy = true
          this.form.post('/vuedataviewer/api/customers')
            .then( response => {
                this.getData()
                $('#customerModal').modal('hide')
                if(this.form.successful)
                {
                  this.$Progress.finish()
                  this.$snotify.success('Customer Successfully Saved.', 'success')
                }
                else
                {
                  this.$Progress.fail()
                  this.$snotify.error('Something Went Wrong !', 'error')
                }
                
            })
            .catch( e=> {
              this.$Progress.fail()
              console.log(e)
            })
        },
        edit(customer){
          this.editMode = true
          this.form.reset()
          this.form.clear()
          this.form.fill(customer)
          $('#customerModal').modal('show')
        },
        update(){
          this.$Progress.start()
          this.form.busy = true
          this.form.put('/vuedataviewer/api/customers/' + this.form.id)
            .then( response => {
                this.getData()
                $('#customerModal').modal('hide')
                if(this.form.successful)
                {
                  this.$Progress.finish()
                  this.$snotify.success('Customer Successfully Updated.', 'Success')
                }
                else
                {
                  this.$Progress.fail()
                  this.$snotify.error('Something Went Wrong !', 'Error')
                }
                
            })
            .catch( e=> {
              this.$Progress.fail()
              console.log(e)
            })
        },
        detail(customer){
          this.form.reset();
          this.form.fill(customer);
          $('#detailModal').modal('show');
          
        },
        destroy(customer){
          this.$snotify.clear()
          this.$snotify.confirm(
            "You won't be able to recover this data!",
            "Are you sure?",
            {
              showProgressBar: false,
              closeOnClick: false,
              pauseOnHover: true,
              buttons: [
                {
                  text: "Yes",
                  action: toast => {
                    this.$snotify.remove(toast.id);
                    this.$Progress.start()
                    axios.delete('/vuedataviewer/api/customers/' + customer.id)
                      .then( response => {
                        this.getData()
                        this.$Progress.finish()
                        this.$snotify.success('Customer Successfully Deleted.', 'Success')   
                      })
                      .catch( e=> {
                        this.$Progress.fail()
                        console.log(e)
                      })
                  },
                 bold: true
                },
                 
                {
                  text: "No",
                  action: toast => {
                    this.$snotify.remove(toast.id);
                  },
                  bold: true
                }
              ]
            }
          );
        },

      }  
    }
</script>
