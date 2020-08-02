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
            <label for="company">company (enter short name)</label>
            <vue-bootstrap-typeahead
              class="mb-4"
              placeholder="Enter short name"
              id="company"
              v-model="company_name"
              v-on:hit="populateLocations"
              :data="entity_list"
            />
          </b-col>
          <b-col cols="2"></b-col>
          <b-col cols="4">
            <label for="location">select location</label>
            <vue-bootstrap-typeahead
              placeholder="Enter location"
              id="location"
              v-model="company_location"
              :data="entity_locations"
              :serializer="item => item.name"
            />
          </b-col>
        </b-row>
        <b-col cols="4">
          <label for="manager">select manager</label>
          <vue-bootstrap-typeahead
            placeholder="Enter manager name"
            id="manager"
            v-model="manager_name"
            :data="emp_list"
            :serializer="item => item.emp_name"
          />
        </b-col>

        <b-row></b-row>
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
  computed: {
    nameState() {
      return this.user_name.length > 0 ? true : false;
    },
    emailState() {
      return this.user_email.length > 0 ? true : false;
    }
  },
  data() {
    return {
      user_name: "",
      user_email: "",
      company_name: "",
      company_location: "",
      entity_list: [],
      entity_locations: [],
      emp_list: [],
      manager_name: "",
      message: "user added succesfully"
    };
  },
  created() {
    /* 1) fetch the companies registered for leave application 
       so that we can show in type ahead 
       2) Also fetch employees so that manager can be selected 
      */
     this.fetchEntities()
     .then(this.fetchEmployees())
     .catch(error=> { console.log("created error "+error)})
     
  },
  methods: {
    fetchEntities: function() {
      return new Promise((resolve, reject) => {
        let query = `
          query entity {
          leave_app_entity {
            name
            short_name
          }
        }`;
        let variables = {};
        const url = process.env.VUE_APP_GRAPHQL_URL;
        const opts = {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ query: query, variables: variables })
        };

        fetch(url, opts)
          .then(res => res.json())
          .then(result => {
            if (
              result &&
              result.data &&
              result.data.leave_app_entity &&
              result.data.leave_app_entity.length > 0
            ) {
              for (let entity of result.data.leave_app_entity) {
                this.entity_list.push(entity.short_name);
              }
              console.log("entity_list " + JSON.stringify(this.entity_list));
              resolve(this.entity_list);
            } else if (result.error) {
              console.log(result.error);
              reject(result.error);
            }
          })
          .catch(error => {
            console.log(error);
            reject(error);
          });
      });
    },
    fetchEmployees: function() {
        return new Promise((resolve, reject) => {
        let query = `
          query all_employees {
          leave_app_employee {
            emp_name
            id
          }
        }`;
        let variables = {};
        const url = process.env.VUE_APP_GRAPHQL_URL;
        const opts = {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ query: query, variables: variables })
        };

        fetch(url, opts)
          .then(res => res.json())
          .then(result => {
            if (
              result &&
              result.data &&
              result.data.leave_app_employee &&
              result.data.leave_app_employee.length > 0
            ) {
              for (let emp of result.data.leave_app_employee) {
                this.emp_list.push(emp);
              }
              console.log("emp_list " + JSON.stringify(this.emp_list));
              resolve(this.emp_list);
            } else if (result.error) {
              console.log(result.error);
              reject(result.error);
            }
          })
          .catch(error => {
            console.log(error);
            reject(error);
          });
      });
    },
    populateLocations: function(selectedEntity) {
      console.log("inside company value selected " + selectedEntity);
      /* For selected company , we need to select locations .
         fire a graphql query 
         select entity_id , location id we are storing keys into user table 
      */
      let query = `
      query entity_locations($company_short_name : String!) {
        leave_app_entity_location(where: {entity: {short_name: {_eq: $company_short_name}}}) {
          id
          entity_id
          name
          code
        }
      } `;
      // query by selected entity ( short company name)
      let variables = {
        company_short_name: selectedEntity
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
          if (
            result &&
            result.data &&
            result.data.leave_app_entity_location &&
            result.data.leave_app_entity_location.length > 0
          ) {
            for (let loc of result.data.leave_app_entity_location) {
              this.entity_locations.push(loc);
            }
          }
        })
        .catch(error => {
          console.log("error in selecting locations" + error);
        });
    },
    addCompany: function() {}
  }
};
</script>