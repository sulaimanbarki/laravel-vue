<script setup>
import { onMounted, ref, reactive } from 'vue';
import { Form, Field } from 'vee-validate';
import * as yup from 'yup';

const users = ref([]);
const editing = ref(false);
const formValues = ref();
const form = ref(null);

const getUsers = () => {
    axios.get('/api/users')
        .then((response) => {
            users.value = response.data;
        })
        .catch((error) => {
            console.log(error);
        });
}

const editUser = (user) => {
    editing.value = true;
    form.value.resetForm();
    $('#modal-default').modal('show');

    formValues.value = {
        id: user.id,
        name: user.name,
        email: user.email
    };
}

const addUser = () => {
    editing.value = false;
}

const createUserSchema = yup.object({
    name: yup.string().required(),
    email: yup.string().email().required(),
    password: yup.string().required().min(8)
});

const updateUserSchema = yup.object({
    name: yup.string().required(),
    email: yup.string().email().required(),
    password: yup.string().when((password, schema) => {
        if (password[0]) {
            return schema.required().min(8);
        }
    })
});

const createUser = (values, { resetForm }) => {
    axios.post('/api/users', values)
        .then((response) => {
            getUsers();
            $('#modal-default').modal('hide');
            resetForm();
        })
        .catch((error) => {
            console.log(error);
        });
}

const updateUser = (values, { resetForm }) => {
    console.log(values);
    axios.put('/api/users/' + values.id, values)
        .then((response) => {
            getUsers();
            $('#modal-default').modal('hide');
            resetForm();
        })
        .catch((error) => {
            console.log(error);
        });
}

onMounted(() => {
    getUsers();
})
</script>

<template>
    <div class="content-header">
        <div class="container-fluid">
            <div class="row mb-2">
                <div class="col-sm-6">
                    <h1 class="m-0">Users</h1>
                </div>
                <div class="col-sm-6">
                    <ol class="breadcrumb float-sm-right">
                        <li class="breadcrumb-item"><a href="#">Home</a></li>
                        <li class="breadcrumb-item active">Users List</li>
                    </ol>
                </div>
            </div>
        </div>
    </div>


    <div class="content">
        <div class="container-fluid">

            <button @click="addUser" type="button" class="btn btn-primary mb-2" data-toggle="modal" data-target="#modal-default">
                Add new user
            </button>

            <div class="card">
                <div class="card-header">
                    <h3 class="card-title">Bordered Table</h3>
                </div>

                <div class="card-body">
                    <table class="table table-bordered">
                        <thead>
                            <tr>
                                <th style="width: 10px">#</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Registration Date</th>
                                <th>Role</th>
                                <th>Option</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(user, index) in users" :key="user.id">
                                <td>{{ index + 1 }}</td>
                                <td>{{ user.name }}</td>
                                <td>{{ user.email }}</td>
                                <td>-</td>
                                <td>-</td>
                                <td>
                                    <a href="#" @click.prevent="editUser(user)"><i class="fa fa-edit"></i></a>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="card-footer clearfix">
                    <ul class="pagination pagination-sm m-0 float-right">
                        <li class="page-item"><a class="page-link" href="#">«</a></li>
                        <li class="page-item"><a class="page-link" href="#">1</a></li>
                        <li class="page-item"><a class="page-link" href="#">2</a></li>
                        <li class="page-item"><a class="page-link" href="#">3</a></li>
                        <li class="page-item"><a class="page-link" href="#">»</a></li>
                    </ul>
                </div>
            </div>
        </div>
    </div>

    <div class="modal fade" id="modal-default">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h4 class="modal-title">
                        <span v-if="editing">Edit User</span>
                        <span v-else>Add New User</span>
                    </h4>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <Form ref="form" @submit="editing ? updateUser : createUser" :validation-schema="editing ? updateUserSchema : createUserSchema" v-slot="{ errors }" :initial-values="formValues">
                    <div class="modal-body">
                        <!-- name -->
                        <div class="form-group">
                            <label for="exampleInputName">Name</label>
                            <Field name="name" type="text" class="form-control" :class="{ 'is-invalid': errors.name }"
                                id="exampleInputName" placeholder="Enter name" />
                            <span class="invalid-feedback">{{ errors.name }}</span>
                        </div>
                        <div class="form-group">
                            <label for="exampleInputEmail1">Email address</label>
                            <Field name="email" type="email" class="form-control" :class="{ 'is-invalid': errors.email }"
                                id="exampleInputEmail1" placeholder="Enter email" />
                            <span class="invalid-feedback">{{ errors.email }}</span>
                        </div>
                        <!-- name -->
                        <div class="form-group">
                            <label for="exampleInputPassword1">Password</label>
                            <Field name="password" type="password" class="form-control"
                                :class="{ 'is-invalid': errors.password }" id="exampleInputPassword1"
                                placeholder="Password" />
                            <span class="invalid-feedback">{{ errors.password }}</span>
                        </div>
                    </div>

                    <div class="modal-footer justify-content-between">
                        <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
                        <button type="submit" class="btn btn-primary">Save changes</button>
                    </div>
                </Form>
            </div>

        </div>

    </div>
</template>