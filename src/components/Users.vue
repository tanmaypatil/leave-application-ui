<template>
  <div id="users">
    <div id="users">
      <b-card no-body>
        <b-row>
          <b-col cols="6">
            <label for="user-name">user name</label>
            <b-form-input
              id="user-name"
              v-model="user_name"
              type="text"
              required
              :state="nameState"
              placeholder="Enter user name"
            ></b-form-input>
          </b-col>
          <b-col cols="4">
            <label for="user-email">user email</label>
            <b-form-input
              id="company-short-name"
              v-model="user_email"
              type="email"
              required
              :state="emailState"
              placeholder="Enter user email"
            ></b-form-input>
          </b-col>
        </b-row>
        <b-row>
          <b-col cols="4">
            <label for="company">select company</label>
            <vue-bootstrap-typeahead id="company" v-model="company_name" :data="entity_list" />
          </b-col>
             <b-col cols="4">
            <label for="company">select location</label>
            <vue-bootstrap-typeahead id="location" v-model="company_location" :data="['Canada', 'USA', 'Mexico']" />
          </b-col>
        </b-row>

        <b-row>
          <!-- <div class="mt-3">Selected: <strong>{{ locations }}</strong></div> -->
        </b-row>
        <b-row>
          <b-col cols="3">
            <b-button v-on:click="addCompany" class="mt-4" pill variant="primary">Add user</b-button>
          </b-col>
        </b-row>
      </b-card>
      <b-modal id="bv-modal-comp-create" hide-footer>
        <div class="d-block text-center">
          <h3>{{ message }}</h3>
        </div>
        <b-button class="mt-3" block @click="$bvModal.hide('bv-modal-comp-create')">Close Me</b-button>
      </b-modal>
    </div>
  </div>
</template>
<script>
export default {
  name: "users",
  data() {
    return {
      user_name: "",
      user_email: "",
      company_name : "",
      company_location : "",
      entity_list : []
    };
  },
  created () {
      /* fetch the companies registered for leave application 
         so that we can show in type ahead 
      */
     let query = `
     query entity {
       leave_app_entity {
       name
       short_name
       }
     }`;

     let variables = {
     };
    const url = process.env.VUE_APP_GRAPHQL_URL;
    const opts = {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ query: query, variables: variables })
    };
    fetch(url, opts)
      .then(res => res.json())
      .then(result => {
          if(result && result.data && result.data.leave_app_entity && result.data.leave_app_entity.length > 0 ) {
              for(let entity of result.data.leave_app_entity ) {
                 this.entity_list.push(entity.short_name);
              }
              console.log("entity_list "+JSON.stringify(this.entity_list));
          }
          else if (result.error) {
              console.log(result.error);
          }
      })
      .catch(error => {
          console.log(error);
      }) ;


  }
};
</script>