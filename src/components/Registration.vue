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
      let restUser = allUsers.value.filter(user => user !== allUsers.value[edit.value.editIndex])
      if (restUser.some((user) => user.FirstName.trim() === formData.value.FirstName.trim() && user.LastName.trim() === formData.value.LastName.trim() && user.Age === formData.value.Age && user.Gender === formData.value.Gender && JSON.stringify(formData.value.Hobbies.sort()) === JSON.stringify(user.Hobbies.sort()) && user.DateOfBirth === formData.value.DateOfBirth && user.State === formData.value.State && user.City === formData.value.City)) {
        errors.value.sameDataError = "Record already exist!"
      }
      else {
        allUsers.value[edit.value.editIndex] = formData.value
        setLocalStorageData(allUsers.value)
        edit.value.editIndex = -1
        edit.value.isEdit = false
        resetForm()
      }
    }
    else {
      if (allUsers.value?.some((user) => user.FirstName.trim() === formData.value.FirstName.trim() && user.LastName.trim() === formData.value.LastName.trim() && user.Age === formData.value.Age && user.Gender === formData.value.Gender && JSON.stringify(formData.value.Hobbies.sort()) === JSON.stringify(user.Hobbies.sort()) && user.DateOfBirth === formData.value.DateOfBirth && user.State === formData.value.State && user.City === formData.value.City)) {
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
            <label for="firstName">First Name</label>
            <input class="form-control" type="text" name="firstName" id="FirstName" v-model="formData.FirstName" />
            <span class="formError text-danger" id="firstNameError">{{ errors.FirstNameError }}</span>
          </div>
          <div class="form-group mb-3">
            <label for="lastName">Last Name</label>
            <input class="form-control" type="text" id="lastName" name="LastName" v-model="formData.LastName" />
            <span class="formError text-danger" id="lastNameError">{{ errors.LastNameError }}</span>
          </div>
          <div class="row justify-content-between">
            <div class="form-group mb-2 col-sm-6">
              <label for="dateOfBirth">Date Of Birth</label>
              <input id="dateOfBirth" name="DOB" class="form-control" type="date" v-model="formData.DateOfBirth" />
              <span class="formError text-danger" id="dateOfBirthError">{{ errors.DOBError }}</span>
            </div>
            <div class="form-group mb-2 col-sm-6">
              <label for="age">Age</label>
              <input id="age" class="form-control" type="number" v-model="formData.Age" />
              <span class="formError text-danger" id="ageError">{{ errors.AgeError }}</span>
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
                <input class="form-check-input" id="male" name="Gender" value="Male" type="radio"
                  v-model="formData.Gender" /><label class="form-check-label" for="male">Male</label>
              </div>
              <div class="form-check">
                <input class="form-check-input" id="female" name="Gender" value="Female" type="radio"
                  v-model="formData.Gender" /><label class="form-check-label" for="female">Female</label>
              </div>
              <span class="formError text-danger" id="GenderError">{{ errors.GenderError }}</span>
            </div>
          </div>

          <div class="row justify-content-between">
            <div class="form-group mb-3 col-sm-6">
              <label for="stateDropdown">Select a State</label>
              <select  id="stateDropdown" class="form-select" v-model="formData.State" @change="handleStateChange($event.target.value)">
                <option value="" disabled selected>Select a State</option>
                <option :value="state" v-for="state in allStates">{{ state }}</option>
              </select>
              <span class="formError text-danger" id="stateError">{{ errors.StateError }}</span>
            </div>
            <div class="form-group mb-3 col-sm-6">
              <label for="cityDropdown">Select a City</label>
              <select id="cityDropdown" class="form-select" v-model="formData.City">
                <option value="" selected disabled>Select a City</option>
                <option :value="city" v-for="city in allCities">{{ city }}</option>
              </select>
              <span class="formError text-danger" id="cityError">{{ errors.CityError }}</span>
            </div>
          </div>

          <button :id="edit.isEdit ? 'update' : 'submit'" class="btn btn-success text-light">{{ !edit.isEdit ? 'Add' : 'Update' }}</button>
        </form>
      </div>
    </div>
  </div>
  <Table :data="allUsers" :handleEdit="handleEdit" :handleDelete="handleDelete" />
</template>
