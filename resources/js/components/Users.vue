<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminOrAuthor()">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users List</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal()">Add New <i class="fas fa-user-plus fa-fw"></i> </button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Registered At</th>
                    <th>Modify</th>
                  </tr>
                  <tr v-for="user in users.data" :key="user.id">
                    <td>{{user.id}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td><span class="tag tag-success">{{user.type | upText}}</span></td>
                    <td>{{user.created_at | dateFormat}}</td>
                    <td>
                        <a href="#" @click="editModal(user)"><i class="fas fa-edit blue"></i></a> 
                        /
                        <a href="#" @click="deleteUser(user.id)"><i class="fas fa-trash red"></i>  </a>
                    </td>
                  </tr>
                  
                </tbody></table>
              </div>
              <!-- /.card-body -->
              <div class="card-footer">
                <pagination :data="users" @pagination-change-page="getResults"></pagination>
              </div>
            </div>
            <!-- /.card -->
          </div>
        </div>

        <div v-else>
          <not-found></not-found>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNewModal" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                  <div class="modal-header">
                      <h5 class="modal-title" id="addNewLabel">{{editMode? 'Update User\'s info' :'Add New'}}</h5>
                      <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                      <span aria-hidden="true">&times;</span>
                      </button>
                  </div>
                  <form @submit.prevent="(editMode? updateUser() : createUser())" @keydown="form.onKeydown($event)">
                    <div class="modal-body">
                      
                        <div class="form-group">
                          <input v-model="form.name" type="text" name="name"
                          placeholder="name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                          <has-error :form="form" field="name"></has-error>
                        </div>

                        <div class="form-group">
                          <input v-model="form.email" type="email" name="email"
                          placeholder="Email Address"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                          <has-error :form="form" field="email"></has-error>
                        </div>

                        <div class="form-group">
                          <textarea v-model="form.bio" type="text" name="bio" id="bio"
                          placeholder="Short bio for user (Optional)"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                          <has-error :form="form" field="bio"></has-error>
                        </div>

                        <div class="form-group">
                          <select v-model="form.type" type="text" name="type"
                            placeholder="type" id="type"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                            <option value="">Select Usert Role</option>
                            <option value="admin">Admin</option>
                            <option value="user">Standard User</option>
                            <option value="author">Author</option>
                          </select>
                          <has-error :form="form" field="type"></has-error>
                        </div>

                        <div class="form-group">
                          <input v-model="form.password" type="password" name="password"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                          <has-error :form="form" field="password"></has-error>
                        </div>
                      
                      
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                        <button type="submit" class="btn btn-primary">{{editMode? 'Update' :'Create'}}</button>
                    </div>
                  </form>  
                </div>
            </div>
        </div>

    </div>
</template>

<script>
    export default {
        data() {
          return {
            editMode: false,
            users: {},
            form: new Form({
              id: '',
              name: '',
              email: '',
              password: '',
              type: '',
              bio: '',
              photo: ''
            })
          }
        },
        methods: {
          getResults(page = 1) {
            axios.get('api/user?page=' + page)
              .then(response => {
                this.users = response.data;
              });
          },
          loadUsers() {
            if(this.$gate.isAdminOrAuthor()) {
              // axios.get("api/user").then(({ data }) => (this.users = data.data));
              this.getResults();
            }
          },
          newModal() {
            this.editMode = false;
            this.form.clear();
            this.form.reset();
             $('#addNewModal').modal('show');
          },
          editModal(user) {
            this.editMode = true;
            this.form.clear();
            this.form.reset();
            this.form.fill(user);
             $('#addNewModal').modal('show');
          },
          deleteUser(id) {
            swal.fire({
              title: 'Are you sure?',
              text: "You won't be able to revert this!",
              type: 'warning',
              showCancelButton: true,
              confirmButtonColor: '#3085d6',
              cancelButtonColor: '#d33',
              confirmButtonText: 'Yes, delete it!'
            }).then((result) => {
              if (result.value) {

                // send request to the server
                axios.delete('api/user/'+id)
                  .then(( data ) => {
                    console.log("data", data);
                    this.loadUsers();
                    swal.fire(
                      'Deleted!',
                      'User has been deleted.',
                      'success'
                    );
                  })
                  .catch((error) => {
                    console.log("failed");
                    swal.fire("Failed!", "There was something wrong.", "warning");
                  });
                
              }
            })
          },
          createUser() {
             this.$Progress.start()
             this.form.post('api/user')
                .then(({ data }) => { 
                  console.log(data);  
                  Fire.$emit('AfterCreate');
                  this.loadUsers(); 
                  $('#addNewModal').modal('hide');
                  this.$Progress.finish();
                  toast.fire({
                    type: 'success',
                    title: 'User created successfully'
                  }) 
                }, (response) => {
                    this.$Progress.fail();
                })
          },
          updateUser() {
            console.log("on update")
             this.$Progress.start();
             this.form.put('api/user/'+this.form.id)
                .then(({ data }) => { 
                  console.log("update",data);  
                  Fire.$emit('AfterCreate');
                  this.loadUsers(); 
                  $('#addNewModal').modal('hide');
                  this.$Progress.finish();
                  toast.fire({
                    type: 'success',
                    title: 'User updated successfully'
                  }) 
                }, (response) => {
                    this.$Progress.fail();
                })
          }
        },
        created() {
            this.loadUsers();      
            // fire.$on('AfterCreate', () => {
            //   this.loadUsers();
            // })  
            
            //search on header app.js fire event
            Fire.$on('searching', () => {
              let query  = this.$parent.search;
              axios.get('api/findUser?q='+query)
                .then(({ data }) => { 
                    console.log("update",data);  
                    this.users = data;
                    this.$Progress.finish();
                    // toast.fire({
                    //   type: 'success',
                    //   title: 'User updated successfully'
                    // }) 
                  })
                  .catch((error) => {
                      this.$Progress.fail();
                  })
            })  
        }
    }
</script>
