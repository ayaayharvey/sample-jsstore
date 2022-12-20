<template>
  <q-page class="flex flex-center">
    <img
      alt="Quasar logo"
      src="~assets/quasar-logo-vertical.svg"
      style="width: 200px; height: 200px"
    />
    <div>
      <div>
        <input type="text" v-model="filter" placeholder="Search" />
        <button @click="search">Search</button>
      </div>
      <input type="text" v-model="newStudent.name" />
      <input type="text" v-model="newStudent.gender" />
      <input type="text" v-model="newStudent.country" />
      <input type="text" v-model="newStudent.city" />
    </div>
    <q-card>
      RECORDS
      <q-card-section v-for="student in students" :key="student?.id">
        <div class="flex">
          <div>{{ student?.name }}</div>
          <div>{{ student?.gender }}</div>
          <div>{{ student?.country }}</div>
          <div>{{ student?.city }}</div>
        </div>
      </q-card-section>
    </q-card>
    <button @click="add">Add</button>
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
