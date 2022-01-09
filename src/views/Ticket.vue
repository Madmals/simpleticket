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
        @click:append="addTask"
        @keyup.enter ="addTask"
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

              <v-list-item-action @click.stop="deleteTicket(index)">
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
// const DB_NAME = "tickets";
// const DB_VER = 1;

export default {
  name: "Ticket",
  data() {
    return {
      newTicket:"",
      settings: "",
      // db:null,
      tickets: [
        {
          title: "ssd:data recovery request",
          done: false,
        },
        { title: "power-supply: kaput customer cakap", done: false },
      ],
    };
  },
  // async created(){
  //   this.db = await this.getDB()
  //   this.ticket = await this.getTicketfromDB()
  // },

  methods: {
    addTask(){
      this.tickets.unshift({title:this.newTicket, done:false})
    },
    doneTicket(id) {
      let ticket = this.tickets.filter((ticket) => ticket.id === id)[0];
      ticket.done = !ticket.done;
    },
    deleteTicket(idx) {
      this.tickets.splice(idx, 1);
    },
  },

  // async addTicket(){
  //   let ticket ={
  //     this.ticket
  //   }
  // }
};
</script>
