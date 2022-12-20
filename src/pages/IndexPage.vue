<template>
  <q-page class="q-pa-md row items-start q-gutter-md">
    <q-card style="width: 400px">
      <q-card-section class="flex">
        <q-input type="text" v-model="filter" placeholder="Search" />
        <q-btn @click="search" icon="search">Search</q-btn>
      </q-card-section>
      <q-card-section>
        <div class="text-h6">ADD FORM</div>
      </q-card-section>
      <q-card-section>
        <q-input type="text" v-model="newStudent.name" placeholder="Name" />
        <q-input type="text" v-model="newStudent.gender" placeholder="Gender" />
        <q-input
          type="text"
          v-model="newStudent.country"
          placeholder="Country"
        />
        <q-input type="text" v-model="newStudent.city" placeholder="City" />
        <q-btn @click="add" class="q-mt-md" icon="add"> Add</q-btn>
      </q-card-section>
    </q-card>

    <q-card flat bordered style="width: 400px">
      <q-card-section>
        <div class="text-h6">RECORDS</div>
      </q-card-section>

      <q-card-section>
        <q-markup-table>
          <thead>
            <tr>
              <th class="text-left">Name</th>
              <th class="text-right">Gender</th>
              <th class="text-right">Country</th>
              <th class="text-right">City</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="student in students" :key="student?.id">
              <td class="text-left">{{ student?.name }}</td>
              <td class="text-right">{{ student?.gender }}</td>
              <td class="text-right">{{ student?.country }}</td>
              <td class="text-right">{{ student?.city }}</td>
            </tr>
          </tbody>
        </q-markup-table>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { Connection, DATA_TYPE } from "jsstore";
import workerInjector from "jsstore/dist/worker_injector";

const workerPath = ref(null);
const connection = new Connection();
const students = ref([]);
const newStudent = ref({
  name: null,
  gender: null,
  country: null,
  city: null,
});
const filter = ref("");

onMounted(() => {
  // workerPath.value = getWorkerPath().default;
  // connection.value = new Connection(new Worker(workerPath));
  connection.addPlugin(workerInjector);
  beforeCreate();
  refreshStudent();
  console.log("students", students);
});

const getWorkerPath = () => {
  // return dev build when env is development
  if (process.env.NODE_ENV === "development") {
    return require("file-loader?name=scripts/[name].[hash].js!jsstore/dist/jsstore.worker.js");
  } else {
    // return prod build when env is production
    return require("file-loader?name=scripts/[name].[hash].js!jsstore/dist/jsstore.worker.min.js");
  }
};

const beforeCreate = async () => {
  try {
    const isDbCreated = await initJsStore();
    if (isDbCreated) {
      console.log("db created");
      // prefill database
    } else {
      console.log("db opened");
    }
  } catch (ex) {
    console.error(ex);
    alert(ex.message);
  }
};

const refreshStudent = async () => {
  students.value = await getStudents();
};

const search = async () => {
  students.value = await getFilter();
};

const add = () => {
  try {
    var studentsAdded = addStudent(
      JSON.parse(JSON.stringify(newStudent.value))
    );
    students.value.push(studentsAdded);
    clear();
  } catch (ex) {
    alert(ex.message);
  }
};

const getStudents = () => {
  return connection.select({
    from: "Students",
  });
};

const getFilter = () => {
  return connection.select({
    from: "Students",
    where: {
      name: { like: "%" + filter.value + "%" },
    },
  });
};

const addStudent = (student) => {
  return connection.insert({
    into: "Students",
    values: [student],
    return: true,
  });
};

const clear = () => {
  newStudent.value = {
    name: "",
    gender: "",
    country: "",
    city: "",
  };
};

const initJsStore = async () => {
  const dataBase = getDatabase();
  return await connection.initDb(dataBase);
};

const getDatabase = () => {
  const tblStudent = {
    name: "Students",
    columns: {
      id: {
        primaryKey: true,
        autoIncrement: true,
      },
      name: {
        notNull: true,
        dataType: DATA_TYPE.String,
      },
      gender: {
        dataType: DATA_TYPE.String,
        default: "male",
      },
      country: {
        notNull: true,
        dataType: DATA_TYPE.String,
      },
      city: {
        dataType: DATA_TYPE.String,
        notNull: true,
      },
    },
  };
  const dataBase = {
    name: "sample",
    tables: [tblStudent],
  };
  return dataBase;
};
</script>
