<template>
  <div id="companies">
    <b-card no-body>
      <b-row>
        <b-col cols="6">
          <label for="company-name">company name</label>
          <b-form-input
            id="company-name"
            v-model="company_name"
            type="text"
            required
            :state="nameState"
            placeholder="Enter company name"
          ></b-form-input>
        </b-col>
        <b-col cols="4">
          <label for="company-short-name">company short name</label>
          <b-form-input
            id="company-short-name"
            v-model="company_short_name"
            type="text"
            required
            :state="shortNameState"
            placeholder="Enter company short name"
          ></b-form-input>
        </b-col>
      </b-row>
      <b-row>
        <b-col cols="4">
          <label for="company-locations">select locations (multiple)</label>
          <b-form-select v-model="locations" :options="options" :state="locationState" multiple :select-size="4"></b-form-select>
        </b-col>
      </b-row>

      <b-row>
        <!-- <div class="mt-3">Selected: <strong>{{ locations }}</strong></div> -->
      </b-row>
      <b-row>
        <b-col cols="3">
          <b-button v-on:click="addCompany" class="mt-4" pill variant="primary">Add company</b-button>
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
</template>
<script>
export default {
  name: "companies",
   computed: {
      nameState() {
        return this.company_name.length > 0 ? true : false
      },
      shortNameState() {
        return this.company_short_name.length > 0 ? true : false
      },
      locationState() {
        return this.locations.length > 0 ? true: false
      }
  },
  data() {
    return {
      company_name: "",
      company_short_name: "",
      options: [
        { value: "PUN", text: "Pune" },
        { value: "CHN", text: "Chennai" },
        { value: "BLR", text: "Bangalore" },
        { value: "MUM", text: "Mumbai" },
        { value: "DEL", text: "Delhi" },
        { value: "KOL", text: "Kolkatta" },
        { value: "NAG", text: "Nagpur" },
        { value: "HYD", text: "Hyderabad" }
      ],
      locations: [],
      message: "company record created successfully"
    };
  },
  methods: {
    addCompany: function() {
      console.log("call the mutation to insert entity and locations");
      console.log("company name " + this.company_name);
      console.log("company locations " + this.locations);

      let inputArr = [];
      let inputObj = {};
      inputObj.name = this.company_name;
      inputObj.short_name = this.company_short_name;
      inputObj.entity_locations = {};
      inputObj.entity_locations.data = [];
      // search through options and populate description for company locations
      for (let loc of this.locations) {
        for (let availLoc of this.options) {
          if (loc == availLoc.value) {
            let obj = {};
            obj.code = loc;
            obj.name = availLoc.text;
            inputObj.entity_locations.data.push(obj);
            break;
          }
        }
      }
      inputArr.push(inputObj);
      // call mutation now to insert company and its locations
      const query = `
      mutation leave_entity_and_locations(
	       $entity_locs: [leave_app_entity_insert_input!]!) {
	       insert_leave_app_entity(objects: $entity_locs) {
            affected_rows
		        returning {
			         id
		        }
	        }
      }`;

      let variables = {
        entity_locs: inputArr
      };

      console.log("vars "+JSON.stringify(inputArr));

      const url = process.env.VUE_APP_GRAPHQL_URL;
      const opts = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ query: query, variables: variables })
      };

      fetch(url, opts)
        .then(res => res.json())
        .then(result => {
           console.log("query result "+JSON.stringify(result));
          if (
            result &&
            result.data &&
            result.data.insert_leave_app_entity.affected_rows >= 1
          ) {
            console.log("company record created");
            // show company created successfully
            this.$bvModal.show("bv-modal-comp-create");
          }
          else if (result.error) {
            console.log("error "+result.error);
          }
        })
        .catch(error => {
          console.log(error);
        });
    }
  }
};
</script>