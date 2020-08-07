<template>
  <div class="container">
    <div class="col-sm-10">
      <h1 class="d-flex justify-content-center mt-4">Задачи</h1>
      <confirmation :message="confirmationMessage"
      v-if="showConfirmation" @showAlert="show_alert"></confirmation>

      <div class="d-flex justify-content-around flex-wrap mb-5 mt-5">
        <h5 class="text-success">Выполнено - {{taskCompleted}} </h5>
        <h5 class="text-danger">Не выполнено - {{taskNotCompleted}} </h5>
      </div>

      <button type="button" id="task-add" class="btn btn-success btn-md align-left d-block"
      v-b-modal.todo-modal> Добавить задачу
      </button>

      <table class="table table-dark table-stripped table-hover">
        <thead class="thead-light">
          <tr>
            <th>Uid</th>
            <th>Описание</th>
            <th>Выполнена?</th>
            <th></th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(todo, index) in todos" :key="index">
            <td class="todo-uid">{{ todo.uid }}</td>
            <td >{{ todo.description }}</td>
            <td>
              <span v-if="todo.is_completed">Выполнено</span>
              <span v-else>Не выполнено</span>
            </td>
            <td>
              <div class="btn-group" role="group">
                <button type="button" class="btn btn-secondary btn-sm" v-b-modal.todo-update-modal
                @click="updateTodo(todo)"> Обновить
                </button>
                &nbsp;
                <button type="button" class="btn btn-danger btn-sm" @click="deleteTodo(todo)">
                  X
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>

      <b-modal ref="addTodoModal" id="todo-modal" title="Добавить задачу" hide-footer>
        <b-form @submit="onSubmit" @reset="onReset" class="w-100">
          <b-form-group id="form-description-group" label="Описание:"
          label-for="form-description-input">
            <b-form-input id="form-description-input" type="text" v-model="addTodoForm.description"
            required placeholder="Завести задачу">
            </b-form-input>
          </b-form-group>
          <b-form-group id="form-complete-group">
            <b-form-checkbox-group v-model="addTodoForm.is_completed" id="form-checks">
              <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
            </b-form-checkbox-group>
          </b-form-group>
          <b-button class="mr-2" type="submit" variant="primary">Добавить</b-button>
          <b-button type="reset" variant="danger">Сброс</b-button>
        </b-form>
      </b-modal>

      <b-modal ref="updateTodoModal" id="todo-update-modal" title="Обновить" hide-footer>
        <b-form @submit="onUpdateSubmit" @reset="onUpdateReset" class="w-100">
        <b-form-group id="form-update-description-group" label="Описание:"
        label-for="form-update-description-input">
          <b-form-input id="form-update-description-input" type="text"
          v-model="updateTodoForm.description" required>
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-update-complete-group">
          <b-form-checkbox-group v-model="updateTodoForm.is_completed" id="form-update-checks">
            <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
          </b-form-checkbox-group>
        </b-form-group>
        <b-button-group>
          <b-button class="mr-2" type="submit" variant="primary">Обновить</b-button>
          <b-button type="reset" variant="danger">Сброс</b-button>
        </b-button-group>
        </b-form>
      </b-modal>

    </div>
  </div>
</template>

<script>
import Confirmation from './Confirmation.vue';

export default {
  name: 'Todo',
  data() {
    return {
      todos: [],
      addTodoForm: {
        description: '',
        is_completed: [],
      },
      updateTodoForm: {
        uid: 0,
        description: '',
        is_completed: [],
      },
      confirmationMessage: '',
      showConfirmation: false,
      showDismissibleAlert: false,
      taskNotCompleted: 0,
      taskCompleted: 0,
      check: false,
    };
  },
  methods: {
    getTodos() {
      // Вызываем функцию подсчет выполненых и невыполненых задач
      this.countTodo();
      // Если нет сохраненных в localStorage задач обнуляем id
      if (localStorage.length <= 2) {
        localStorage.setItem('id', 0);
      }
      for (let i = 0; i < localStorage.length; i += 1) {
        const itemsArr = localStorage.getItem(i) ? JSON.parse(localStorage.getItem(i)) : [];
        if (itemsArr.length !== 0) {
          this.todos.push(itemsArr);
        }
      }
    },
    resetForm() {
      this.addTodoForm.description = '';
      this.addTodoForm.is_completed = [];
      this.updateTodoForm.description = '';
      this.updateTodoForm.is_completed = [];
    },
    onSubmit(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();

      const requestData = {
        uid: localStorage.getItem('id'),
        description: this.addTodoForm.description,
        is_completed: this.addTodoForm.is_completed.length > 0,
      };

      localStorage.setItem('id', Number(localStorage.id) + 1);

      const requestDataJson = JSON.stringify(requestData);
      localStorage.setItem(requestData.uid, requestDataJson);

      this.todos = [];
      this.getTodos();

      this.confirmationMessage = `Задача "${requestData.description}" добавлена`;
      this.showConfirmation = true;
      this.resetForm();
    },
    onReset(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();
      this.resetForm();
    },
    updateTodo(todo) {
      // Добавить проверку на наличие задачи с таким id в localStorage
      this.updateTodoForm.uid = todo.uid;
      this.updateTodoForm.description = todo.description;
      if (todo.is_completed) {
        this.updateTodoForm.is_completed = [true];
      }
    },
    onUpdateSubmit(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      const requestData = {
        uid: this.updateTodoForm.uid,
        description: this.updateTodoForm.description,
        is_completed: this.updateTodoForm.is_completed.length > 0,
      };

      this.checkTodo(requestData);
      if (this.check) {
        const requestDataJson = JSON.stringify(requestData);
        localStorage.setItem(requestData.uid, requestDataJson);

        this.todos = [];
        this.getTodos();

        this.confirmationMessage = `Задача "${requestData.uid}" обновлена`;
        this.showConfirmation = true;
        this.resetForm();
      } else {
        this.todos = [];
        this.getTodos();
        this.confirmationMessage = `Задача с id "${requestData.uid}" была удалена ранее. 
        Не возможно ее обновить.`;
        this.showConfirmation = true;
        this.resetForm();
      }
    },
    onUpdateReset(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      this.resetForm();
    },
    deleteTodo(todo) {
      localStorage.removeItem(todo.uid);

      for (let i = this.todos.length - 1; i >= 0; i -= 1) {
        if (this.todos[i].uid === todo.uid) {
          this.todos.slice(i, i);
        }
      }

      this.todos = [];
      this.getTodos();

      this.confirmationMessage = `Задача "${todo.description}" удалена из списка`;
      this.showConfirmation = true;
    },
    countTodo() {
      this.taskNotCompleted = 0;
      this.taskCompleted = 0;
      for (let i = 0; i < localStorage.length; i += 1) {
        const itemsArr = localStorage.getItem(i) ? JSON.parse(localStorage.getItem(i)) : [];
        if (itemsArr.is_completed === true) {
          this.taskCompleted += 1;
        }
        if (itemsArr.is_completed === false) {
          this.taskNotCompleted += 1;
        }
      }
    },
    checkTodo(requestData) {
      let count = 0;
      for (let i = 0; i < localStorage.length; i += 1) {
        if (requestData.uid === localStorage.key(i)) {
          count += 1;
        }
      }
      if (count > 0) {
        this.check = true;
      } else {
        this.check = false;
      }
    },
    show_alert() {
      this.showConfirmation = !this.showConfirmation;
    },
  },
  components: {
    confirmation: Confirmation,
  },
  created() {
    localStorage.setItem('id', 0);
    this.getTodos();
  },
};
</script>

<style>
  button#task-add {
   margin-top: 20px;
   margin-bottom: 20px;
  }
  h1, td {
   text-align: left;
  }
  .todo-uid {
   text-align: right;
  }
</style>
