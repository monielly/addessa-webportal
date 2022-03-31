<template>
  <div class="flex column">
    <div id="_wrapper" class="pa-5">
      <v-overlay :absolute="absolute" :value="overlay">
        <v-progress-circular
          :size="70"
          :width="7"
          color="primary"
          indeterminate
        ></v-progress-circular>
      </v-overlay>
      <v-main>
        <v-breadcrumbs :items="items">
          <template v-slot:item="{ item }">
            <v-breadcrumbs-item :to="item.link" :disabled="item.disabled">
              {{ item.text.toUpperCase() }}
            </v-breadcrumbs-item>
          </template>
        </v-breadcrumbs>
        <v-card>
          <v-card-title class="mb-0 pb-0">
            <span class="headline">Create User</span>
          </v-card-title>
          <v-divider></v-divider>
          <v-card-text class="pa-6">
            <v-row>
              <v-col cols="4" class="mt-0 mb-0 pt-0 pb-0">
                <v-text-field
                  name="event_name"
                  v-model="editedItem.event_name"
                  label="Event Name"
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <v-data-table
                  :headers="headers"
                  :items="expense_particulars"
                  :hide-default-header="true"
                  :hide-default-footer="true"
                >
                  <!-- <template slot="no-data">
                    <div> add item</div>
                  </template> -->
                  <template v-slot:header="{ props }">
                    <thead class="grey darken-1 white--text font-weight-bold">
                      <tr>
                        <td>PARTICULARS</td>
                        <td width="110px"></td>
                      </tr></thead
                  ></template>
                  <template v-slot:item.description="{ item, index }">
                    <v-text-field
                      name="description"
                      v-model="item.description"
                      dense
                      hide-details
                      outlined
                      class="mt-2 mb-2"
                    ></v-text-field>
                  </template>
                  <template v-slot:item.actions="{ item, index }">
                    <v-btn color="primary" @click="addItem()" icon>
                      <v-icon>mdi-plus-circle</v-icon>
                    </v-btn>
                    <v-btn color="red" icon @click="removeItem(item)">
                      <v-icon class="ma-0 pa-0">mdi-minus-circle</v-icon>
                    </v-btn>
                  </template>
                  <template v-slot:footer="{ props }">
                    <tfoot class="pa-0">
                      <tr class="pa-0">
                        <td class="pa-0 text-right" colspan="2">
                          <v-btn
                            color="primary"
                            class="mt-3"
                            @click="addItem()"
                            small
                          >
                            add item
                          </v-btn>
                        </td>
                      </tr>
                    </tfoot>
                  </template>
                </v-data-table>
                <!-- <v-simple-table>
                  <thead class="grey darken-1 white--text font-weight-bold">
                    <tr>
                      <td>PARTICULARS</td>
                      <td width="110px"></td>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(item, index) in expense_particulars">
                      <td>
                        <v-text-field
                          name="description"
                          v-model="item.description"
                          dense
                          hide-details
                          outlined
                          class="mt-2 mb-2"
                        ></v-text-field>
                      </td>
                      <td>
                        <v-btn color="red" icon @click="removeItem(item)">
                          <v-icon class="ma-0 pa-0">mdi-minus-circle</v-icon>
                        </v-btn>
                        <v-btn color="primary" @click="addItem()" icon>
                          <v-icon>mdi-plus-circle</v-icon>
                        </v-btn>
                      </td>
                    </tr>
                  </tbody>
                  <tfoot>
                    <tr>
                      <td class="pa-2 text-right" colspan="2">
                        <v-btn color="primary" @click="addItem()" small>
                          add item
                        </v-btn>
                      </td>
                    </tr>
                  </tfoot>
                </v-simple-table> -->
                <v-list>
                  <v-list-group
                    v-for="item in items"
                    :key="item.title"
                    v-model="item.active"
                    :prepend-icon="item.action"
                    no-action
                  >
                    <template v-slot:activator>
                      <v-list-item-content>
                        <v-list-item-title
                          v-text="item.title"
                        ></v-list-item-title>
                      </v-list-item-content>
                    </template>

                    <v-list-item v-for="child in item.items" :key="child.title">
                      <v-list-item-content>
                        <v-list-item-title
                          v-text="child.title"
                        ></v-list-item-title>
                      </v-list-item-content>
                    </v-list-item>
                  </v-list-group>
                </v-list>
              </v-col>
            </v-row>
          </v-card-text>

          <v-card-actions>
            <v-btn
              color="primary"
              @click="save"
              :disabled="disabled"
              class="ml-4 mb-4 mr-1"
            >
              Save
            </v-btn>
            <v-btn color="#E0E0E0" to="/" class="mb-4"> Cancel </v-btn>
          </v-card-actions>
        </v-card>
      </v-main>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import { validationMixin } from "vuelidate";
import {
  required,
  maxLength,
  email,
  minLength,
  sameAs,
} from "vuelidate/lib/validators";

export default {
  mixins: [validationMixin],

  validations: {
    editedItem: {
      event_name: { required },
    },
  },
  data() {
    return {
      absolute: true,
      overlay: false,
      items: [
        {
          text: "Home",
          disabled: false,
          link: "/",
        },
        {
          text: "Expense Particular Lists",
          disabled: false,
          link: "/tactical_requisition/expense_particular/index",
        },
        {
          text: "Create Expense Particular",
          disabled: true,
        },
      ],
      headers: [
        { text: "Description", value: "description" },
        { text: "Actions", value: "actions" },
      ],
      disabled: false,

      editedItem: {
        name: "",
      },
      defaultItem: {
        name: "",
      },

      expense_particulars: [],
    };
  },

  methods: {
    showAlert() {
      this.$swal({
        position: "center",
        icon: "success",
        title: "Record has been saved",
        showConfirmButton: false,
        timer: 2500,
      });
    },

    save() {
      this.$v.$touch();
      this.userError = {
        name: [],
        email: [],
        password: [],
        confirm_password: [],
        branch_id: [],
      };

      if (!this.$v.$error) {
        this.disabled = true;
        this.overlay = true;

        const data = this.editedItem;

        axios.post("/api/user/store", data).then(
          (response) => {
            if (response.data.success) {
              // send data to Sockot.IO Server
              // this.$socket.emit("sendData", { action: "user-create" });

              this.showAlert();
              this.clear();

              //push recently added data from database
              this.users.push(response.data.user);
            } else {
              let errors = response.data;
              let errorNames = Object.keys(response.data);

              errorNames.forEach((value) => {
                this.userError[value].push(errors[value]);
              });
            }
            this.overlay = false;
            this.disabled = false;
          },
          (error) => {
            this.isUnauthorized(error);

            this.overlay = false;
            this.disabled = false;
          }
        );
      }
    },
    clear() {
      this.$v.$reset();
      this.editedItem = Object.assign({}, this.defaultItem);
    },

    isUnauthorized(error) {
      // if unauthenticated (401)
      if (error.response.status == "401") {
        this.$router.push({ name: "unauthorize" });
      }
    },
    addItem() {
      this.expense_particulars.push({ description: "" });
    },
    removeItem(item) {
      let index = this.expense_particulars.indexOf(item);
      this.expense_particulars.splice(index, 1);
    },
  },
  computed: {
    nameErrors() {
      const errors = [];
      if (!this.$v.editedItem.name.$dirty) return errors;
      !this.$v.editedItem.name.required && errors.push("Name is required.");
      return errors;
    },
  },
  mounted() {
    axios.defaults.headers.common["Authorization"] =
      "Bearer " + localStorage.getItem("access_token");
  },
};
</script>