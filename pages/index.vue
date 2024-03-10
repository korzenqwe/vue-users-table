<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-row>
          <v-col cols="12" sm="6" md="4" lg="3">
            <v-text-field label="Цвет" v-model="colorFilter"></v-text-field>
          </v-col>
          <v-col cols="12" sm="6" md="4" lg="3">
            <v-text-field label="Роль" v-model="roleFilter"></v-text-field>
          </v-col>
          <v-col cols="12" sm="6" md="4" lg="3">
            <v-switch label="Блокировка" v-model="blockedFilter"></v-switch>
          </v-col>
          <v-col cols="12" sm="6" md="4" lg="3">
            <v-range-slider
              v-model="ageRange"
              :min="0"
              :max="100"
              thumb-label="always"
              label="Возраст"
            ></v-range-slider>
          </v-col>
        </v-row>

        <v-data-table
          :headers="headers"
          :items="filteredUsers"
          :search="search"
          :items-per-page="10"
          class="elevation-1"
        >
          <template v-slot:top>
            <v-toolbar flat>
              <v-toolbar-title>Пользователи</v-toolbar-title>
              <v-spacer></v-spacer>
              <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                label="Поиск"
                single-line
                hide-details
              ></v-text-field>
            </v-toolbar>
          </template>
          <template v-slot:item.color="{ item }">
            <v-chip :color="item.color" dark>{{ item.color }}</v-chip>
          </template>
          <template v-slot:item.blocked="{ item }">
            <v-chip :color="item.blocked ? 'red' : 'green'" dark>{{
              item.blocked ? "Заблокирован" : "Активен"
            }}</v-chip>
          </template>
        </v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      search: "",
      colorFilter: "",
      roleFilter: "",
      blockedFilter: false,
      ageRange: [0, 100],
      headers: [
        { text: "ID", value: "id" },
        { text: "Имя", value: "name" },
        { text: "Баланс", value: "balance" },
        { text: "Цвет", value: "color" },
        { text: "Роль", value: "role" },
        { text: "Блокировка", value: "blocked" },
        { text: "Возраст", value: "age" },
      ],
      users: [],
    };
  },

  computed: {
    filteredUsers() {
      return this.users.filter((user) => {
        return (
          (!this.colorFilter || user.color === this.colorFilter) &&
          (!this.roleFilter || user.role === this.roleFilter) &&
          (this.blockedFilter === null ||
            user.blocked === this.blockedFilter) &&
          this.ageRange[0] <= user.age &&
          user.age <= this.ageRange[1]
        );
      });
    },
  },

  async fetch() {
    this.colorFilter = this.$route.query.color || "";
    this.roleFilter = this.$route.query.role || "";
    this.blockedFilter = this.$route.query.blocked || false;
    this.ageRange = [
      this.$route.query.ageMin || 0,
      this.$route.query.ageMax || 100,
    ];

    try {
      this.users = (await this.$axios.get("/api/users"))?.data || [];
    } catch (error) {
      console.error(error);
    }
  },

  watch: {
    colorFilter(newVal) {
      this.$router.push({
        query: { ...this.$route.query, color: newVal || undefined },
      });
    },
    roleFilter(newVal) {
      this.$router.push({
        query: { ...this.$route.query, role: newVal || undefined },
      });
    },
    blockedFilter(newVal) {
      this.$router.push({
        query: { ...this.$route.query, blocked: newVal || undefined },
      });
    },
    ageRange(newVal) {
      this.$router.push({
        query: {
          ...this.$route.query,
          ageMin: newVal[0] || undefined,
          ageMax: newVal[1] || undefined,
        },
      });
    },
  },
};
</script>

<style scoped>
.v-input--range-slider {
  margin-top: 16px;
}
</style>
