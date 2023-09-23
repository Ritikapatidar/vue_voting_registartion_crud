<script setup>
import { ref, onMounted } from 'vue';
import validations from '../composables/validation';
import Table from './Table.vue';
import stateCity from '../assets/stateCities.json'
const { validate } = validations()
const formData = ref({
  FirstName: '', LastName: '', DateOfBirth: '', Gender: '', Age: '', Hobbies: [], State: '', City: ''
})
const errors = ref({
  FirstNameError: '', LastNameError: '', DOBError: '', GenderError: '', AgeError: '', HobbiesError: '', StateError: '', CityError: '', sameDataError: ''
})
const allUsers = ref([])
const allStates = ref([])
const allCities = ref([])
const edit = ref({ isEdit: false, editIndex: -1 })

onMounted(
  () => {
    allStates.value = Object.keys(stateCity).sort()
    allUsers.value = JSON.parse(localStorage.getItem('voting_userData'))
  }
)
const handleStateChange = (e) => {
  formData.value.City = ''
  allCities.value = stateCity[e].sort()
}

const setLocalStorageData = (data) => {
  localStorage.setItem('voting_userData', JSON.stringify(data))
}
const resetForm = () => {
  formData.value = {
    FirstName: '', LastName: '', DateOfBirth: '', Gender: '', Age: '', Hobbies: [], State: '', City: ''
  }
}
function handleSubmit() {
  if (validate(formData.value, errors.value)) {

    errors.value.sameDataError = ''
    if (edit.value.isEdit) {
      allUsers.value[edit.value.editIndex] = formData.value
      setLocalStorageData(allUsers.value)
      edit.value.editIndex = -1
      edit.value.isEdit = false
      resetForm()
    }
    else {
      if (allUsers.value.some((user) => JSON.stringify(user) === JSON.stringify(formData.value))) {
        errors.value.sameDataError = "Record already exist!"
      }
      else {
        if (localStorage.getItem('voting_userData')) {
          setLocalStorageData([formData.value, ...JSON.parse(localStorage.getItem('voting_userData'))])
        }
        else {
          setLocalStorageData([formData.value])
        }
        resetForm()
      }
    }
    allUsers.value = JSON.parse(localStorage.getItem('voting_userData'))
  }
}

const handleDelete = (index) => {
  allUsers.value = allUsers.value.filter((user) => allUsers.value.indexOf(user) !== index)
  setLocalStorageData(allUsers.value)
}

const handleEdit = (index) => {
  edit.value.isEdit = true
  edit.value.editIndex = index
  const editItem = allUsers.value[index]

  formData.value.FirstName = editItem.FirstName
  formData.value.LastName = editItem.LastName
  formData.value.DateOfBirth = editItem.DateOfBirth
  formData.value.Gender = editItem.Gender
  formData.value.Age = editItem.Age
  formData.value.Hobbies = editItem.Hobbies
  formData.value.State = editItem.State
  handleStateChange(editItem.State)
  formData.value.City = editItem.City
}

onMounted(() => {
  allUsers.value = JSON.parse(localStorage.getItem('voting_userData'))
})
</script>
<template>
  <div class="container my-2 m-auto">
    <div class="row py-3">
      <h2 class="mx-2">Voting Registration Form</h2>
      <div class="col-12">
        <form @submit.prevent="handleSubmit" @change="validate(formData, errors)" class="p-3 rounded border">
          <span class="formError text-danger" id="SameDataError">{{ errors.sameDataError }}</span>
          <div class="form-group mb-3">
            <label for="FirstName">First Name</label>
            <input class="form-control" type="text" name="FirstName" id="FirstName" v-model="formData.FirstName" />
            <span class="formError text-danger" id="FirstNameError">{{ errors.FirstNameError }}</span>
          </div>
          <div class="form-group mb-3">
            <label for="LastName">Last Name</label>
            <input id="LastName" class="form-control" type="text" name="LastName" v-model="formData.LastName" />
            <span class="formError text-danger" id="LastNameError">{{ errors.LastNameError }}</span>
          </div>
          <div class="row justify-content-between">
            <div class="form-group mb-2 col-6">
              <label for="DOB">Date Of Birth</label>
              <input id="DOB" name="DOB" class="form-control" type="date" v-model="formData.DateOfBirth" />
              <span class="formError text-danger" id="DOBError">{{ errors.DOBError }}</span>
            </div>
            <div class="form-group mb-2 col-6">
              <label for="Age">Age</label>
              <input id="Age" class="form-control" type="number" v-model="formData.Age" />
              <span class="formError text-danger" id="AgeError">{{ errors.AgeError }}</span>
            </div>
          </div>
          <div class="row justify-content-between">
            <div class="form-group mb-3 col-6">
              <label for="Hobbies">Hobbies</label>
              <div class="form-check">
                <input class="form-check-input" name="Hobbies" type="checkbox" v-model="formData.Hobbies" value="Chess"
                  id="Chess">
                <label class="form-check-label" for="Chess">
                  Chess
                </label>
              </div>
              <div class="form-check">
                <input class="form-check-input" name="Hobbies" type="checkbox" v-model="formData.Hobbies"
                  value="Badminton" id="Badminton">
                <label class="form-check-label" for="Badminton">
                  Badminton
                </label>
              </div>
              <div class="form-check">
                <input class="form-check-input" name="Hobbies" type="checkbox" v-model="formData.Hobbies" value="Hockey"
                  id="Hockey">
                <label class="form-check-label" for="Hockey">
                  Hockey
                </label>
              </div>
              <div class="form-check">
                <input class="form-check-input" name="Hobbies" type="checkbox" v-model="formData.Hobbies" value="Reading"
                  id="Reading">
                <label class="form-check-label" for="Reading">
                  Reading
                </label>
              </div>
              <span class="formError text-danger" id="HobbiesError">{{ errors.HobbiesError }}</span>
            </div>
            <div class="form-group mb-2 col-6">
              <label for="Gender">Gender</label>
              <div class="form-check">
                <input class="form-check-input" id="Male" name="Gender" value="Male" type="radio"
                  v-model="formData.Gender" /><label class="form-check-label" for="Male">Male</label>
              </div>
              <div class="form-check">
                <input class="form-check-input" id="Female" name="Gender" value="Female" type="radio"
                  v-model="formData.Gender" /><label class="form-check-label" for="Female">Female</label>
              </div>
              <span class="formError text-danger" id="GenderError">{{ errors.GenderError }}</span>
            </div>
          </div>

          <div class="form-group mb-3">
            <label for="State">Select a State</label>
            <select class="form-select" v-model="formData.State" @change="handleStateChange($event.target.value)">
              <option value="" disabled selected>Select a State</option>
              <option :value="state" v-for="state in allStates">{{ state }}</option>
            </select>
            <span class="formError text-danger" id="StateError">{{ errors.StateError }}</span>
          </div>
          <div class="form-group mb-3">
            <label for="City">Select a City</label>
            <select class="form-select" v-model="formData.City">
              <option value="" selected disabled>Select a City</option>
              <option :value="city" v-for="city in allCities">{{ city }}</option>
            </select>
            <span class="formError text-danger" id="CityError">{{ errors.CityError }}</span>
          </div>

          <button id="btnSubmit" class="btn btn-success text-light">{{ !edit.isEdit ? 'Add' : 'Update' }}</button>
        </form>
      </div>
    </div>
  </div>
  <Table :data="allUsers" :handleEdit="handleEdit" :handleDelete="handleDelete" />
</template>