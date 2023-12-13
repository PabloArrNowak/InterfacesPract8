<script setup>
import { ref, computed } from "vue";
import { resolve } from "../model.js";

// basado en https://vuejs.org/examples/#grid

const props = defineProps({
  data: Array,
  columns: Array,
  filterKey: String,
});
defineEmits(["choose"]);

const sortKey = ref("");
const sortOrders = ref(props.columns.reduce((o, key) => ((o[key] = 1), o), {}));

const filteredData = computed(() => {
  let { data, filterKey } = props;
  if (filterKey) {
    filterKey = filterKey.toLowerCase();
    data = data.filter((row) => {
      return Object.keys(row).some((key) => {
        return String(row[key]).toLowerCase().indexOf(filterKey) > -1;
      });
    });
  }
  const key = sortKey.value;
  if (key) {
    const order = sortOrders.value[key];
    data = data.slice().sort((a, b) => {
      a = a[key];
      b = b[key];
      return (a === b ? 0 : a > b ? 1 : -1) * order;
    });
  }
  return data;
});

function sortBy(key) {
  sortKey.value = key;
  sortOrders.value[key] *= -1;
}

function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}
</script>

<template>
  <table v-if="filteredData.length">
    <thead>
      <tr>
        <th
          v-for="key in columns"
          :key="key"
          @click="sortBy(key)"
          :class="{
            active: sortKey == key,
            'special-cell': [
              'name',
              'members',
              'ram',
              'groups',
              'state',
            ].includes(key),
          }"
        >
          {{ capitalize(key) }}
          <span
            v-if="sortKey == key"
            :class="`arrow ${sortOrders[key] > 0 ? 'asc' : 'dsc'}`"
          >
          </span>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="entry in filteredData"
        :key="entry.id"
        @click="$emit('choose', entry.id)"
      >
        <td
          v-for="key in columns"
          :key="`_${entry.id}_${key}`"
          class="text-start"
        >
          <template v-if="key === 'name'">
            <span class="name">{{ entry[key] }}</span>
          </template>
          <template v-else-if="Array.isArray(entry[key])">
            <td>
              <template
                v-for="vm in entry[key].slice(0, 4)"
                :key="`_${vm.id}_${vm}`"
              >
                <span class="badge bg-primary">{{ resolve(vm).name }}</span>
                <!--________________INTENTO DE SEPARAR POR COLORES POR ESTADO_____________
                <template v-if="resolve(vm).state === RUNNING">
                  <span class="badge bg-success">{{ resolve(vm).name }}</span>
                </template>
                <template v-else-if="resolve(vm).state === STOPPED">
                  <span class="badge bg-danger">{{ resolve(vm).name }}</span>
                </template>
                <template v-else-if="resolve(vm).state === SUSPENDED">
                  <span class="badge bg-warning text-dark">{{ resolve(vm).name }}</span>
                </template>
                -->
              </template>
              <template v-if="entry[key].length > 4">
                <span
                  class="badge bg-info"
                  :title="
                    entry[key]
                      .slice(4)
                      .map((v) => resolve(v).name)
                      .join(', ')
                  "
                  >+{{ entry[key].length - 4 }}</span
                >
              </template>
            </td>
          </template>
          <template v-else>
            {{ entry[key] }}
          </template>
        </td>
      </tr>
    </tbody>
  </table>
  <p v-else>(no hay nada que mostrar)</p>
</template>

<style>
.arrow.asc::after {
  content: "↓";
}
.arrow.dsc::after {
  content: "↑";
}
span.name {
  font-weight: 1000;
}
table {
  margin-top: 10px;
  display: table;
  border-collapse: separate;
  border-spacing: 2px;
  border-color: gray;
}
thead > tr {
  border-bottom: 1px solid gray;
  color: rgb(104, 103, 103);
}

th.special-cell {
  background-color: rgba(144, 144, 144, 0.166);
  color: black;
  border: 2px solid rgba(144, 144, 144, 0.829);
  transition-duration: 0.4s;
  box-shadow: 0 4px 16px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

th.special-cell:hover {
  background-color: rgba(144, 144, 144, 0.829);
  color: rgb(255, 254, 254);
  box-shadow: 0 6px 16px 0 rgba(0, 0, 0, 0.24),
    0 17px 50px 0 rgba(0, 0, 0, 0.19);
}

/* td:nth-child(even) {
  background-color: rgba(150, 212, 212, 0.4);
} */
tr:nth-child(even) {
  background-color: #d6eeee;
}
</style>
