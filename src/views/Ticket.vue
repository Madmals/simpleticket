<template>
  <div>
    <v-divider></v-divider>

    <v-list subheader flat class="pt-0">
      <v-subheader>Ticket Lists</v-subheader>
      <v-text-field
        v-model="newTicket"
        class="pa-3"
        outlined
        hide-details
        filled
        clearable
        @click:append="addnewTicket"
        @keyup.enter="addnewTicket"
        label="Add Ticket"
        append-icon="mdi-sticker-plus-outline"
      ></v-text-field>
      <v-list-item-group v-model="settings" multiple>
        <div v-for="(detail, index) in tickets" :key="index">
          <v-list-item
            @click="doneTicket(detail.id)"
            :class="{ 'green lighten-3': detail.done }"
          >
            <!-- this will give bg blue to done up for class -->
            <template v-slot:default="">
              <v-list-item-action>
                <v-checkbox
                  :input-value="detail.done"
                  color="primary"
                ></v-checkbox>
              </v-list-item-action>

              <v-list-item-content>
                <v-list-item-title
                  :class="{ 'text-decoration-line-through': detail.done }"
                  >{{ detail.title }}</v-list-item-title
                >
              </v-list-item-content>

              <!-- add .stop in order prevent line throught happen too  -->

              <v-list-item-action @click.stop="dels(detail.id)">
                <v-btn icon>
                  <v-icon color="red lighten-1">mdi-delete-circle</v-icon>
                </v-btn>
              </v-list-item-action>
            </template>
          </v-list-item>
          <v-divider></v-divider>
        </div>
      </v-list-item-group>
    </v-list>
  </div>
</template>

<script>
const DB_NAME = "tickets";
const DB_VER = 1;

export default {
  name: "Ticket",
  data() {
    return {
      newTicket: null,
      settings: "",
      db: null,
      tickets: [],
    };
  },
  async created() {
    this.db = await this.getDb();
    this.tickets = await this.getTicketfromDb()
  },

  methods: {
    addTicket() {
      this.tickets.unshift({ title: this.newTicket, done: false });
    },
    doneTicket(id) {
      let ticket = this.tickets.filter((ticket) => ticket.id === id)[0];
      ticket.done = !ticket.done;
    },
    deleteTicket(idx) {
      this.tickets.splice(idx, 1);
    },

    async addnewTicket() {
      if (this.newTicket) {
        let ticket = { title: this.newTicket, done: false };
        await this.addTickettoDb(ticket);
        this.tickets = await this.getTicketfromDb();
      } else {
        return;
      }
    },
    async dels(id) {
      await this.deleteTicketfromDb(id);
      this.tickets = await this.getTicketfromDb();
    },

    async addTickettoDb(ticket) {
      return new Promise((resolve, reject) => {
        let tx = this.db.transaction(["tickets"], "readwrite");
        tx.oncomplete = () => {
          resolve();
        };
        tx.onerror = () => {
          reject();
        };
        let store = tx.objectStore("tickets");
        store.add(ticket);

      });
    },

    async deleteTicketfromDb(id) {
      return new Promise((resolve, reject) => {
        let tx = this.db.transaction(["tickets"], "readwrite");
        tx.oncomplete = (e) => {
          resolve(e);
        };
        let store = tx.objectStore("tickets");
        store.delete(id);

        tx.onerror = () => {
          reject();
        };
      });
    },

    async getTicketfromDb() {
      return new Promise((resolve, reject) => {
        let tx = this.db.transaction(["tickets"], "readonly");
        tx.oncomplete = () => {
          resolve(tickets); //must return a value from below
        };

        tx.onerror = () => {
          reject();
        };

        let store = tx.objectStore("tickets");
        let tickets = [];


        //get store object to iterate or shown
        store.openCursor().onsuccess = (e) => {
          let cursor = e.target.result;
          if (cursor) {
            tickets.push(cursor.value);
            cursor.continue();
          }
        };
      });
    },
    async getDb() {
      return new Promise((resolve, reject) => {
        let req = window.indexedDB.open(DB_NAME, DB_VER);

        req.onupgradeneeded = (e) => {
          console.log("onupgradeneeded");
          let db = e.target.result;

          db.createObjectStore("tickets", {
            keyPath: "id",
            autoIncrement: true,
          });
        };

        req.onsuccess = (e) => {
          resolve(e.target.result);
        };

        req.onerror = () => {
          reject("error");
        };
      });
    },
  },
};
</script>
