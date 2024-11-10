<template>
  <v-app>
    <v-container class="pa-5" style="background-color: #f3f4f6; min-height: 100vh;">
      <v-card class="pa-5" outlined elevation="3" style="border-radius: 12px;">
        <v-card-title class="text-h4 text-center">Dompetku v2 – Expense & Income Money Management</v-card-title>
        <v-divider></v-divider>

        <!-- Form for new entry -->
        <v-form @submit.prevent="addEntry">
          <v-row>
            <v-col cols="12" sm="4">
              <v-text-field v-model="newEntry.description" label="Keterangan" outlined required></v-text-field>
            </v-col>
            <v-col cols="12" sm="4">
              <v-text-field v-model.number="newEntry.amount" label="Jumlah Uang" type="number" outlined required></v-text-field>
            </v-col>
            <v-col cols="12" sm="4">
              <v-select
                v-model="newEntry.category"
                :items="categories"
                label="Kategori"
                outlined
                required
              ></v-select>
            </v-col>
            <v-col cols="12" class="text-center">
              <v-btn type="submit" color="primary" class="mt-3">Tambah Data</v-btn>
            </v-col>
          </v-row>
        </v-form>

        <!-- Summary -->
        <v-card-subtitle class="mt-5">
          <strong>Summary:</strong><br>
          Total Entries: {{ entries.length }}<br>
          Total Income: Rp {{ totalIncome }}<br>
          Total Expense: Rp {{ totalExpense }}<br>
          Balance: Rp {{ balance }}
        </v-card-subtitle>

        <!-- List of entries -->
        <v-list class="mt-5">
          <v-subheader class="text-h6">Rekap Keuangan</v-subheader>
          <v-alert v-if="entries.length === 0" type="info">Belum ada data.</v-alert>
          <v-list-item
            v-for="(entry, index) in entries"
            :key="index"
            class="mb-2"
          >
            <v-list-item-content>
              <v-list-item-title>{{ entry.description }} - Rp {{ entry.amount }} ({{ entry.category }})</v-list-item-title>
            </v-list-item-content>
            <v-list-item-action>
              <v-btn icon color="red" @click="removeEntry(index)">
                <v-icon>mdi-delete</v-icon>
              </v-btn>
            </v-list-item-action>
          </v-list-item>
        </v-list>

        <!-- Clear all entries button -->
        <v-btn color="error" class="mt-5" @click="confirmClearAll">
          Hapus Semua Data
        </v-btn>
      </v-card>
    </v-container>
  </v-app>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import { loadEntries, saveEntries, clearEntries } from '../stores/store.js';

export default {
  data() {
    return {
      newEntry: {
        description: "",
        amount: null,
        category: "",
      },
      categories: ["Food", "Transport", "Shopping", "Other"],
      entries: loadEntries()
    };
  },
  computed: {
    totalIncome() {
      return this.entries
        .filter(entry => entry.amount > 0)
        .reduce((sum, entry) => sum + entry.amount, 0);
    },
    totalExpense() {
      return this.entries
        .filter(entry => entry.amount < 0)
        .reduce((sum, entry) => sum + Math.abs(entry.amount), 0);
    },
    balance() {
      return this.totalIncome - this.totalExpense;
    }
  },
  methods: {
    addEntry() {
      if (this.newEntry.description && this.newEntry.amount && this.newEntry.category) {
        this.entries.push({ ...this.newEntry });
        saveEntries(this.entries);
        this.newEntry = { description: "", amount: null, category: "" };
      } else {
        alert("Harap isi keterangan, jumlah uang, dan kategori.");
      }
    },
    removeEntry(index) {
      this.entries.splice(index, 1);
      saveEntries(this.entries);
    },
    confirmClearAll() {
      if (confirm("Apakah Anda yakin ingin menghapus semua data?")) {
        this.clearAllEntries();
      }
    },
    clearAllEntries() {
      this.entries = [];
      clearEntries();
    }
  },
  onMounted() {
    this.entries = loadEntries();
  }
};
</script>

<style scoped>
.v-container {
  background-color: #f3f4f6;
  min-height: 100vh;
}

.v-card {
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.v-btn {
  font-weight: bold;
  border-radius: 8px;
}

.v-alert {
  color: #757575;
}
</style>
