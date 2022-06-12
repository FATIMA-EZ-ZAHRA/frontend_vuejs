<template>
  <v-container fluid>
    <div class="tables-basic">
      <h1 class="page-title mt-10 mb-6">Gestion des Conférences</h1>

      <v-data-table :headers="headers" :items="conferences" sort-by="titre" class="elevation-1">
        <template v-slot:top>
          <v-toolbar flat>
            <v-toolbar-title>Liste des Conférences</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-dialog v-model="dialog" max-width="1000px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                  Nouveau
                  <v-icon right dark> mdi-plus </v-icon>
                </v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="text-h5">{{ formTitle }}</span>
                </v-card-title>

                <v-card-text>
                  <v-container>
                    <v-form ref="form" v-model="valid" lazy-validation>
                      <v-row>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field v-model="editedItem.titre" label="Titre Conférence"></v-text-field>
                        </v-col>

                        <v-col cols="12" sm="6" md="4">
                          <v-menu ref="menu1" v-model="menu1" :close-on-content-click="false" :return-value.sync="date"
                            transition="scale-transition" offset-y min-width="auto">
                            <template v-slot:activator="{ on, attrs }">
                              <v-text-field v-model="editedItem.date_debut" label="Date Debut"
                                prepend-icon="mdi-calendar" readonly v-bind="attrs" v-on="on"></v-text-field>
                            </template>
                            <v-date-picker v-model="editedItem.date_debut" :rules="date_debutRules" no-title scrollable>
                              <v-spacer></v-spacer>
                              <v-btn text color="primary" @click="menu1 = false">
                                Annuler
                              </v-btn>
                              <v-btn text color="primary" @click="$refs.menu1.save(date)">
                                Valider
                              </v-btn>
                            </v-date-picker>
                          </v-menu>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-menu ref="menu2" v-model="menu2" :close-on-content-click="false" :return-value.sync="date"
                            transition="scale-transition" offset-y min-width="auto">
                            <template v-slot:activator="{ on, attrs }">
                              <v-text-field v-model="editedItem.date_fin" label="Date Fin" prepend-icon="mdi-calendar"
                                readonly v-bind="attrs" v-on="on"></v-text-field>
                            </template>
                            <v-date-picker v-model="editedItem.date_fin" :rules="date_finRules" no-title scrollable>
                              <v-spacer></v-spacer>
                              <v-btn text color="primary" @click="menu2 = false">
                                Annuler
                              </v-btn>
                              <v-btn text color="primary" @click="$refs.menu2.save(date)">
                                Valider
                              </v-btn>
                            </v-date-picker>
                          </v-menu>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field v-model="editedItem.frais" label="Frais"></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">

                          <v-select :items="amphitheatres" item-text="nom" item-value="id" 
                            :rules="amphiRules" label="Amphiteathre">

                          </v-select>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-select :items="conferenciers" item-text="nom" item-value="id"
                            :rules="conferencierRules" label="Conférencier">

                          </v-select>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">

                          <v-textarea v-model="editedItem.description" label="Déscription"></v-textarea>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">

                          <v-textarea v-model="editedItem.details" label="Détails"></v-textarea>
                        </v-col>
                      </v-row>
                    </v-form>
                  </v-container>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="close">
                    Annuler
                  </v-btn>
                  <v-btn color="blue darken-1" text @click="save">
                    Enregistrer
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
            <v-dialog v-model="dialogDelete" max-width="700px">
              <v-card>
                <v-card-title class="text-h7 text-center">voulez-vous vraiment supprimer cet élément ?</v-card-title>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="closeDelete">Annuler</v-btn>
                  <v-btn color="red darken-1" text @click="deleteItemConfirm">Oui, Supprimer</v-btn>
                  <v-spacer></v-spacer>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>
        <template v-slot:[`item.actions`]="{ item }">
          <div class="text-center">
            <v-menu offset-y>
              <template v-slot:activator="{ on, attrs }">
                <v-btn color="primary" dark v-bind="attrs" v-on="on">

                  <v-icon>
                    mdi-cog-outline
                  </v-icon>
                  Actions
                </v-btn>
              </template>
              <v-list>
                <v-list-item @click="showItem(item)">
                  <v-list-item-title>Consulter
                    <v-icon>
                      mdi-eye
                    </v-icon>
                  </v-list-item-title>
                </v-list-item>
                <v-list-item @click="editItem(item)">
                  <v-list-item-title>Modifier
                    <v-icon>
                      mdi-pencil
                    </v-icon>
                  </v-list-item-title>
                </v-list-item>
                <v-list-item @click="deleteItem(item)">
                  <v-list-item-title>Supprimer
                    <v-icon>
                      mdi-delete
                    </v-icon>
                  </v-list-item-title>
                </v-list-item>

              </v-list>
            </v-menu>
          </div>
        </template>
        <template v-slot:no-data>
          <v-btn color="primary" @click="initialize"> Reset </v-btn>
        </template>
      </v-data-table>
    </div>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  data: () => ({
    dialog: false,
    dialogDelete: false,
    headers: [
      {
        text: "ID",
        align: "start",
        sortable: false,
        value: "id",
      },
      { text: "Titre", value: "titre" },
      //{ text: "Conférencier", value: "conferencier.nom" },
      { text: "Déscription", value: "description" },
     // { text: "Détails", value: "details" },
      { text: "Date Debut", value: "date_debut" },
      { text: "Date Fin", value: "date_fin" },
      //{ text: "Frais", value: "frais" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    conferences: [],
    editedIndex: -1,
    deletedID: -1,
    titreRules: [
      v => !!v || 'Le titre est Obligatoire',

    ],
    descriptionRules: [
      v => !!v || 'La déscription est Obligatoire',

    ],
    detailsRules: [
      v => !!v || 'Le détails est Obligatoire',

    ],
    date_debutRules: [
      v => !!v || 'La date de début est Obligatoire',
    ],
    date_finRules: [
      v => !!v || 'La date de fin est Obligatoire',
    ],
    fraisRules: [
      v => !!v || 'Le frais est Obligatoire',
      v => (v && Number.isInteger(Number(v))) || 'Le cout doit être un nombre.',
    ],
    amphiRules: [
      v => !!v || "L'amphitheatre est Obligatoire",
    ],
    conferencierRules: [
      v => !!v || "Le conférencier est Obligatoire",
    ],
    editedItem: {
      titre: "",
      description: "",
      details: "",
      date_debut: "",
      date_fin: "",
      frais: "",
      //amphi_id: "",
     // conferencier_id: "",

    },
    valid: true,
    defaultItem: {
      titre: "",
      description: "",
      details: "",
      date_debut: "",
      date_fin: "",
      frais: "",
      //amphi_id: "",
      //conferencier_id: "",
    },
    menu1: false,
    menu2: false,
    date: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),

    errors: [],
    amphitheatres: [],
    conferenciers: [],
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1
        ? "Ajouter Conference"
        : "Modifier Conference";
    },

  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.initialize();
  },

  methods: {
    initialize() {
      let url = "conferences/";
      axios
        .get(url)
        .then((response) => {
          console.log(response);
          this.conferences = response.data;
        })
        .catch((e) => {
          this.errors.push(e.data);
          console.log(e);
        });

      this.getAmphis();
      this.getConferenciers();

    },
    getAmphis() {
      let url = "amphitheatres/";
      axios
        .get(url)
        .then((response) => {
          console.log(response);
          this.amphitheatres = response.data;
        })
        .catch((e) => {
          this.errors.push(e);
          console.log(e);
        });
    },
    getConferenciers() {
      let url = "conferenciers/";
      axios
        .get(url)
        .then((response) => {
          console.log(response);
          this.conferenciers = response.data;
        })
        .catch((e) => {
          this.errors.push(e);
          console.log(e);
        });
    },
    editItem(item) {
      this.editedIndex = item.id;
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.deletedID = item.id;

      //this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      let url = "conferences/";
     
      axios
        .delete(url + this.deletedID + "/")
        .then((response) => {
          console.log(response);
          this.showMessage("Elément Supprimé avec succès.");
          this.initialize();
        })
        .catch((e) => {
          this.errors.push(e);
          console.log(e);
        });
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
console.log() 

      if (this.validate()) {

        this.editedItem.date_debut = (new Date(this.editedItem.date_debut)).toISOString().split('T')[0]
        this.editedItem.date_fin = (new Date(this.editedItem.date_fin)).toISOString().split('T')[0]
      console.log(">>"+this.editedItem.date_debut);
        let url = "conferences/";
        if (this.editedIndex > -1) {
          axios
            .put(url + this.editedItem.id + "/", this.editedItem)
            .then((response) => {
              console.log(response);
              this.showMessage("Elément Modifié avec succès.");
              this.initialize();
            })
            .catch((e) => {
              this.errors.push(e);
              console.log(e);
            });
        } else {
          axios
            .post(url, this.editedItem)
            .then((response) => {
              console.log(response);
              this.showMessage("Elément Enregistré avec succès.");
              this.initialize();
            })
            .catch((e) => {
              this.errors.push(e.response.data);
              console.log(e.response.data);
            });

          // this.conferences.push(this.editedItem);
        }
        this.close();

      }
    },
    showMessage(msg) {
      this.$toast.success(msg);
    },
    showItem(item) {
      this.$router.push({ name: 'DetailsConference', params: { id: item.id } });
    },
    validate() {
      return this.$refs.form.validate()
    },

    resetValidation() {
      this.$refs.form.resetValidation()
    },
  },
};
</script>

<style src="./Basic.scss" lang="scss">
</style>
