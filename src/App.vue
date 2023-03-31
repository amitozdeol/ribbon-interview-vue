<template>
  <v-app>
    <v-app-bar>
      <v-container class="d-flex align-center py-0">
        <v-app-bar-title class="pl-0">
          <div class="d-flex align-center">Ribbon</div>
        </v-app-bar-title>
      </v-container>
    </v-app-bar>

    <v-main>
      <section id="hero">
        <v-sheet class="d-flex align-center pb-16" color="grey-darken-3">
          <v-container class="text-center">
            <v-responsive class="mx-auto">
              <h3 class="text-h3">Try Ribbon's all new features</h3>

              <p class="mt-4 text-medium-emphasis">
                Our all-in-one platform gives you the banking, accounting,
                fundraising, and organizational tools you need to build a
                successful charity under the umbrella of your fiscal sponsor.
              </p>
            </v-responsive>
          </v-container>
        </v-sheet>
      </section>

      <v-sheet>
        <section id="filter">
          <v-container>
            <v-row justify="space-between">
              <v-col cols="auto">
                <h2 class="text-h4">Ribbon Donor List</h2>

                <p class="text-primary mt-3">In Beta now!</p>

                <p class="mt-3">See all those that have given in one place!</p>
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <v-text-field
                  v-model="search"
                  label="Search"
                  append-icon="mdi-magnify"
                  @keyup.native="updateSearch"
                  ref="search"
                  dense
                  outlined
                ></v-text-field>
                <v-progress-linear
                  v-if="!donors"
                  color="success"
                  indeterminate
                  height="4"
                ></v-progress-linear>
                <v-data-table
                  v-else
                  :headers="headers"
                  :items="donors.data"
                  :items-per-page="5"
                  :sort-by="sortBy"
                  :sort-desc="sortDesc"
                  class="table"
                  item-key="name"
                >
                  <template
                    v-slot:item="{ item }"
                    :class="{ 'row-hover': true }"
                    @mouseover="onRowHover(true, item)"
                    @mouseout="onRowHover(false, item)"
                  >
                    <tr
                      :style="{
                        color: 'rgba(58,58,64,0.87)',
                        backgroundColor: 'transparent',
                      }"
                      class="row-hover"
                    >
                      <td>{{ item.full_name }}</td>
                      <td>{{ item.email }}</td>
                      <td>{{ item.total_donations }}</td>
                      <td>{{ formatDate(item.first_donation) }}</td>
                    </tr>
                  </template>
                </v-data-table>

                <table v-if="donors">
                  <thead>
                    <tr>
                      <th class="text-left">Name</th>
                      <th class="text-left">Email</th>
                      <th class="text-left">Total Donations</th>
                      <th class="text-left">First Donation</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="item in donors.data" :key="item.id">
                      <td>{{ item.full_name }}</td>
                      <td>{{ item.email }}</td>
                      <td>{{ item.total_donations }}</td>
                      <td>{{ item.first_donation }}</td>
                    </tr>
                  </tbody>
                </table>
              </v-col>
            </v-row>
          </v-container>
        </section>
      </v-sheet>

      <v-sheet class="py-16" color="#1818181a">
        <section id="grid">
          <v-container>
            <v-row justify="space-between">
              <v-col cols="auto">
                <v-responsive width="350">
                  <h2 class="text-h4">Show your support feature</h2>
                  <p class="text-success mt-3">Available now!</p>
                  <p class="mt-3">
                    Easily send messages to those that have given!
                  </p>
                </v-responsive>
              </v-col>
              <v-sheet width="400" class="mx-auto">
                <v-form
                  v-model="valid"
                  validate-on="submit"
                  @submit.prevent="submit"
                >
                  <v-textarea
                    v-model="message"
                    :rules="messageRules"
                    label="Message"
                  ></v-textarea>

                  <v-autocomplete
                    label="Search for email"
                    v-model="form.email"
                    :items="donors?.data"
                    :loading="!donors"
                    item-text="email"
                    item-value="id"
                    :search-input.sync="searchText"
                    return-object
                    @change="update"
                  >
                  </v-autocomplete>

                  <v-text-field
                    v-model="form.donor_id"
                    label="Donor Id"
                  ></v-text-field>
                  <v-btn type="submit" block class="mt-2">Send</v-btn>
                </v-form>
              </v-sheet>
            </v-row>
          </v-container>
        </section>
      </v-sheet>
    </v-main>

    <v-footer>
      <v-container
        class="text-overline d-flex align-center justify-space-between"
      >
        <div>Copyright &copy; 2023 Flourish Change Inc dba Ribbon</div>

        <v-icon icon="mdi-bank" size="x-large" />
      </v-container>
    </v-footer>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  name: "App",

  data() {
    return {
      search: "",
      sortBy: "name",
      sortDesc: false,
      headers: [
        { text: "Name", value: "name", sortable: true },
        { text: "Email", value: "email", sortable: true },
        {
          text: "Total Donations",
          value: "total_donations",
          align: "right",
          sortable: true,
        },
        {
          text: "First Donation",
          value: "first_donation",
          align: "right",
          sortable: true,
        },
      ],
      donors: null,
      fixed_donors: null,
      valid: false,
      searchText: "",
      form: {
        email: "",
        donor_id: "",
      },
      message: "",
      emailRules: [
        (value) => {
          if (value) return true;

          return "E-mail is required.";
        },
      ],
      messageRules: [
        (value) => {
          if (value) return true;

          return "Message is required.";
        },
      ],
    };
  },
  watch: {
    async searchText(val) {
      try {
        if (val?.length >= 3) {
          this.donors = null;
          const response = await fetch(
            `https://interview.ribbon.giving/api/donors?search=${this.searchText}`,
            {
              headers: { "Content-Type": "application/json" },
            }
          );
          const data = await response.json();
          this.donors = data;
        }
      } catch (error) {
        console.error(error);
      }
    },
  },
  mounted() {
    axios.get("https://interview.ribbon.giving/api/donors").then((response) => {
      this.donors = response.data;
      this.fixed_donors = response.data.data;
    });
  },
  methods: {
    async submit() {
      // Send message to server.
      // make a post request to https://interview.ribbon.giving/api/donors/{donor_id}/send-message
      try {
        const response = await fetch(
          `https://interview.ribbon.giving/api/donors/${this.form.donor_id}/send-message`,
          {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ message: this.message }),
          }
        );
        const data = await response.json();
        console.log(data);
      } catch (error) {
        console.error(error);
      }
    },
    update(e) {
      //update form data values
      this.form.email = e.email;
      this.form.donor_id = e.id;
    },

    formatDate(dateString) {
      const date = new Date(dateString);
      const options = {
        year: "numeric",
        month: "short",
        day: "numeric",
        hour: "numeric",
        minute: "numeric",
        second: "numeric",
        hour12: false,
      };
      return new Intl.DateTimeFormat("en-US", options).format(date);
    },
    updateSearch() {
      this.$nextTick(() => {
        this.search = this.$refs.search.value;
        this.donors.data = this.fixed_donors.filter((donor) => {
          return donor.full_name
            .toLowerCase()
            .includes(this.search.toLowerCase());
        });
      });
    },
  },
};
</script>
<style>
.v-data-table-header {
  background-color: rgb(58, 58, 64, 0.05);
}
.row-hover:hover {
  background-color: rgba(58, 58, 64, 0.03) !important;
}
</style>
