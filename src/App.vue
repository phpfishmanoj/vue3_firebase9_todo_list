<template>
  <div class="baddas-todo">
    <div class="title has-text-centered">
      <h1>Todo List.....</h1>
    </div>
    <form @submit.prevent="addTodo">
      <div class="field is-grouped mb-5">
        <p class="control is-expanded">
          <input v-model="addTodoContent" class="input" type="text" placeholder="Add Todo">
        </p>
        <p class="control">
          <button :disabled="!addTodoContent" class="button is-info">
            Add
          </button>
        </p>
      </div>
    </form>
    <div v-for="tlist in  todos" :class="{ 'has-background-success-light': tlist.done }" class="card mb-5">
      <div class="card-content">
        <div class="content">
          <div class="columns is-mobile is-vcentered">
            <div :class="{ 'has-text-success line-through': tlist.done }" class="column">
              {{ tlist.content }}
            </div>
            <div class="column is-5 has-text-right">
              <button @click="toggleDone(tlist.id)" :class="tlist.done ? 'is-success' : 'is-light'" class="button">
                &check;
              </button>
              <button @click="delteToDo(tlist.id)" class="button is-danger ml-2">
                &cross;
              </button>
            </div>
          </div>

        </div>
      </div>
    </div>

  </div>
</template> 

<script setup>
/*
imports
*/
import { ref, onMounted } from "vue";
import { v4 as uuidv4 } from "uuid";
//import { collection, getDocs } from "firebase/firestore";
import {
  collection, onSnapshot, addDoc, doc, updateDoc,
  deleteDoc, query, orderBy, limit
} from "firebase/firestore"; // for realtime data
import { db } from "./components/firebase/db.js";

/*Firebase ref*/

const todosCollectionRef = collection(db, 'toloCollection');
const todosCollectionQuery = query(todosCollectionRef, orderBy("date", "desc"), limit(2));

/*  Todos */
const todos = ref([
  // {
  //   id: 1,
  //   content: 'Laravel Practice',
  //   done: false,
  // },
  // {
  //   id: 2,
  //   content: 'Vue Practice',
  //   done: true,
  // },
  // {
  //   id: 3,
  //   content: 'React Practice',
  //   done: false,
  // }
]);

/* get todos from firebase*/

onMounted(async () => {
  /* 
  // for await we are using to need to add async
  onMounted(async () => {   
  const querySnapshot = await getDocs(collection(db, 'toloCollection'));
   let fbTodos = [];
   querySnapshot.forEach((res) => {
     // res.data() is never undefined for query doc snapshots
     //console.log(res.id, " => ", res.data());
     const todo = {
       id: res.id,
       content: res.data().content,
       done: res.data().done,
     }
     fbTodos.push(todo);
   }); //foreach completed
   todos.value = fbTodos; */

  onSnapshot(todosCollectionQuery, (querySnapshot) => {
    const fbTodos = [];
    querySnapshot.forEach((res) => {
      const todo = {
        id: res.id,
        content: res.data().content,
        done: res.data().done,
      }
      fbTodos.push(todo);
    });//foreach completed
    todos.value = fbTodos;
  });

});

/* add Todo */
const addTodoContent = ref('');

const addTodo = () => {
  // const newTodoAdd = {
  //   id: uuidv4(),
  //   content: addTodoContent.value,
  //   done: false,
  // }
  //console.log(newTodoAdd);
  //todos.value.unshift(newTodoAdd);

  addDoc(todosCollectionRef, {
    content: addTodoContent.value,
    done: false,
    date: Date.now(),
  });

  addTodoContent.value = '';
}

/*Toggle Done*/

const toggleDone = (id) => {
  const index = todos.value.findIndex(todo => todo.id === id)
  //const status = todos.value[index].done = !todos.value[index].done;

  updateDoc(doc(todosCollectionRef, id), {
    done: !todos.value[index].done,
  });
}

/* Delete Todo */

const delteToDo = (id) => {
  //console.log(id);
  //todos.value = todos.value.filter(todo => todo.id !== id);
  deleteDoc(doc(todosCollectionRef, id));
}

</script>

<style>
@import 'bulma/css/bulma.min.css';

.baddas-todo {
  max-width: 400;
  padding: 20px;
  margin: 0 auto;
}

.line-through {
  text-decoration: line-through;
}
</style>