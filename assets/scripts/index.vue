// Данный компонент статичен, его логика минимальна - отображение заметок.
// События на кнопках передают экшены в родительский компонент, где они обрабатываются
Vue.component('note', {
  props: ['note'],
  data() {
    return {
      id: this.note.id,
      confirmVisible: false
    }
  },
  template: `
  <div class="note">
    <div class="note__header">
      <p class="note__title">{{note.name}}</p>
      <div class="note__buttons">
        <button v-on:click="confirmVisible = true" class="note__btn note__btn_delete btn" title="Удалить заметку">
          <img src='./assets/images/icon-remove.svg' alt='icon' class='note__icon note__icon_remove'>
        </button>
        <button v-on:click="$emit('edit-note', id)" class="note__btn note__btn_edit btn" title="Редактировать заметку">
          <img src='./assets/images/icon-edit.svg' alt='icon' class='note__icon note__icon_edit'>
        </button>      
      </div>
      <div v-if="confirmVisible" class="note__confirm">
        <p class="note__confirm-question">Уверены, что хотите удалить заметку?</p>
        <div class="note__confirm-buttons">
          <button v-on:click="$emit('delete-note', id)" class="note__confirm-btn note__confirm-btn_yes btn">Да</button>
          <button v-on:click="confirmVisible = false" class="note__confirm-btn note__confirm-btn_no btn">Нет</button>
        </div>
      </div>
    </div>
    <div class="note__tasks">
      <div v-for="(task, index) in note.list" v-if="index <= 1" :key="index" class="note__task">
        <input class="note__task-checkbox" type="checkbox" disabled :checked="task.checked">
        <p class="note__task-name">{{task.task}}</p>
      </div>
      <span v-if="note.list.length > 2" class="note__dots">...</span>
    </div>
  </div>`
})

// При инициализации этого компонента нам потребуются исходные данные, чтобы к ним можно было вернуться,
// отменив ввод, текущие данные - для отображение и изменения, и измененные данные - чтобы отменить отмену ввода.
// События сохраниния-удаления заметки передаются и обрабатываются в родительском компоненте
Vue.component('mutable-note', {
  props: ['note'],
  data() {
    return {
      sourceData: {
        id: this.note.id,
        name: this.note.name,
        list: this.note.list.map(task => ({...task})),
      },
      newData: {
        id: this.note.id,
        name: this.note.name,
        list: this.note.list.map(task => ({...task})),
      },
      editedData: null,
      isEdited: false,
      confirmVisible: false,
      confirmQuestion: null,
      typeOfAction: null,
    }
  },
  mounted() {
    this.$refs.nameInput.focus()
  },
  methods: {
    saveData() {
      return this.$emit('save-data', this.newData);
    },
    addTask() {
      this.newData.list.push({task: '', checked: false})
      setTimeout(() => this.$refs.taskInput[this.$refs.taskInput.length - 1].focus(), 50);
      this.isEdited = true;
    },
    deleteTask(index) {
      this.newData.list.splice(index, 1);
      this.isEdited = true;
    },
    // метод обработки действий, требующих подтверждения. 
    actionHandler(action) {
      this.confirmVisible = true;
      switch (action){
        case 'reset':
          this.confirmQuestion = 'Уверены, что хотите отменить редактирование?';
          this.typeOfAction = 'reset'
          break;
        case 'delete':
          this.confirmQuestion = 'Уверены, что хотите удалить заметку?';
          this.typeOfAction = 'delete'
          break;
      }
    },
    // метод отправки события в родительский компонент, для обработки
    sendAction() {
      switch (this.typeOfAction) {
        case 'reset':
          return this.$emit('reset-edit');
        case 'delete':
          return this.$emit('delete-note', this.newData.id);
      }
    },
    resetInputData(){
      this.editedData = this.newData;
      this.newData = {...this.sourceData};
      this.newData.list = this.sourceData.list.map(task => ({...task}));
      this.isEdited = false;
    },
    returnInputData(){
      if(this.editedData) {
        this.newData = this.editedData;
        this.editedData = null;
        this.isEdited = true;
      }
    },
  },
  template: `
    <div class="note">
      <div class="note__header">
        <input v-model="newData.name" v-on:input="isEdited = true; editedData = null" type="text" class="note__title-input" ref="nameInput"
        />
        <div class="note__buttons">
          <div class="note__wrapper">
            <button v-on:click="saveData" class="note__btn note__btn_save btn" title="Сохранить заметку">
              <img src='./assets/images/icon-save.svg' alt='icon' class='note__icon note__icon_save'>
            </button>
            <button v-on:click="actionHandler('reset')" class="note__btn note__btn_reset btn" title="Отменить редактирование">
              <img src='./assets/images/icon-reset.svg' alt='icon' class='note__icon note__icon_reset'>
            </button>
            <button v-on:click="actionHandler('delete')" class="note__btn note__btn_delete btn" title="Удалить заметку">
              <img src='./assets/images/icon-remove.svg' alt='icon' class='note__icon note__icon_remove'>
            </button>
          </div>
          <div class="note__wrapper">
            <button v-on:click="resetInputData" :disabled="!this.isEdited" class="note__btn note__btn_save btn" title="Отменить введенные данные">
              <img src='./assets/images/icon-left.svg' alt='icon' class='note__icon note__icon_left'>
            </button>
            <button v-on:click="returnInputData" :disabled="!this.editedData" class="note__btn note__btn_reset btn" title="Отменить сброс данных">
              <img src='./assets/images/icon-right.svg' alt='icon' class='note__icon note__icon_right'>
            </button>
          </div>
        </div>
        <div v-if="confirmVisible" class="note__confirm">
          <p class="note__confirm-question">{{confirmQuestion}}</p>
          <div class="note__confirm-buttons">
            <button v-on:click="sendAction" class="note__confirm-btn note__confirm-btn_yes btn">Да</button>
            <button v-on:click="confirmVisible = false" class="note__confirm-btn note__confirm-btn_no btn">Нет</button>
          </div>
        </div>
      </div>
      <div class="note__tasks">
        <div class="note__tasks-buttons">
          <button v-on:click="addTask" class="note__btn note__btn_add-task btn" title="Добавить задачу">
              <img src='./assets/images/icon-add-task.svg' alt='icon' class='note__icon note__icon_add-task'>
          </button>
        </div>
        <div v-for="(task, index) in newData.list" :key="index" class="note__task">
          <input v-on:change="isEdited = true; editedData = null" lass="note__task-checkbox" type="checkbox" v-model="task.checked">
          <input v-on:input="isEdited = true; editedData = null" type="text" v-model="task.task" class="note__task-input" ref="taskInput"/>
          <button v-if="newData.list.length > 1" v-on:click="deleteTask(index)" class="note__btn note__btn_delete-task btn" title="Удалить задачу">
              <img src='./assets/images/icon-delete-task.svg' alt='icon' class='note__icon note__icon_delete-task'>
          </button>
        </div>
      </div>
    </div>`
})

// Корневой родительский компонент. Содержит данные приложения и основные методы их обработки.
new Vue({
  el: '#app',
  created() {
    if(localStorage.notes) this.notes = JSON.parse(localStorage.getItem('notes'));
  },
  data: {
    notes: {},
    mutableNote: null,
  },
  methods: {
    editNote(id) {
      this.mutableNote = {...this.notes[id]};
      this.mutableNote.list = this.notes[id].list.map(task => ({...task}))
    },
    deleteNote(id) {
      const obj = {...this.notes};
      delete obj[id];
      this.notes = obj;
      this.mutableNote = null;
      localStorage.setItem('notes', JSON.stringify(this.notes));
    },
    saveData(data) {
      this.notes = {...this.notes, [data.id]: {...data}};
      localStorage.setItem('notes', JSON.stringify(this.notes));
      this.mutableNote = null;
    },
    addNote() {
      this.mutableNote = {
        id: this.getId(),
        name: '',
        list: [{
          task: '',
          checked: false,
        }],
      }
    },
    // метод ищет среди заметок наибольший id, и возвращает число, увеличенное на единицу
    // для присвоения id новой заметке
    getId() {
      let biggestId = Object.keys(this.notes).sort((a, b) => b - a)[0];
      return biggestId ? Number(++biggestId) : 1;
    },
    resetEdit() {
      this.mutableNote = null;
    }
  }
});