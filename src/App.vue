<script>
  import CheckSquare from './icons/check-square.vue';
  import Square from './icons/square.vue';
  import Pencil from './icons/pencil.vue';
  import Trash from './icons/trash.vue';
  import PlusCircle from './icons/plus-circle.vue';
  import SwitchVertical from './icons/switch-vertical.vue';
  import Circle from './icons/circle.vue';
  import RecordCircle from './icons/record-circle.vue';
  import ChevronRight from './icons/chevron-right.vue';
  import ChevronBottom from './icons/chevron-bottom.vue';
  import axios from 'axios';
  
  let timeout;
  
  export default {
    data() {
      return {
        data: {
          version: 2,
          todos: [],
          user: {
            firstName: undefined,
          },
          imageUrl: '',
        },
        cache: {
          filteredTodos: undefined,
          expandPreferences: false,
          currentContent: '',
          searchContent: '',
          lastDeletedTodo: undefined,
          showLastDeletedTodoNotification: false,
        },
        preferences: {
          sortDesc: false,
          hideBanner: false,
          date: 'hide' /* hide, date, time, full */,
        },
      };
    },
    watch: {
      data: {
        handler(data) {
          localStorage.setItem('data', JSON.stringify(data));
        },
        deep: true,
      },
    },
    computed: {
      /**
       * @returns all filtered todos or filtered todos matching search results
       * @description If any input is entered in the search bar, todos containing the entered input are displayed, otherwise all filtered todos are displayed.
       */
      filterTodos() {
        if (this.cache.searchContent !== '') {
          return this.cache.filteredTodos.filter(todo =>
            todo.content.toLowerCase().includes(this.cache.searchContent.toLowerCase())
          );
        }
  
        return this.cache.filteredTodos;
      },
    },
    created() {
      if (localStorage.getItem('data')) {
        let checkObj = JSON.parse(localStorage.getItem('data'));
  
        switch (checkObj.version) {
          case 2:
            /* v-1.3 and above */
            this.data = checkObj;
            break;
          case 1:
            /* between v-1.1.1 and v-1.2.1 */
            this.data = checkObj;
            this.data.version = 2;
            this.data.user = {};
            checkObj.preferences.hideCompletedTodos
              ? (this.data.preferences.showTodos = 'active')
              : (this.data.preferences.showTodos = 'all');
            break;
          default:
            /* v-1.1.0 and below */
            localStorage.clear();
            break;
        }
      }
      localStorage.setItem('data', JSON.stringify(this.data));
    },
    mounted() {
  
      this.rearrangeTodos();
    },
    methods: {
      /**
       * @param {Number} id - ID property of object in todos array
       * @description - Sets true if the completed property of the selected to-do object is false, false if true.
       */
      toggleStatus(id) {
        this.data.todos.find(todo => todo.id === id).completed = !this.data.todos.find(
          todo => todo.id === id
        ).completed;
      },
      /**
       * @description This method is used to add a todo when the + button or enter is pressed after the user has entered the todo input.
       */
      addTodo() {
        this.cache.currentContent = this.cache.currentContent.trim();
        if (this.cache.currentContent === '') this.cache.currentContent = 'Blank to-do';
  
        let newTodo = {
          id: Math.round(Math.random() * 1000000000000),
          content: this.cache.currentContent,
          date: Date.now(),
          completed: false,
        };
  
        this.unshiftOrPushHandler(newTodo);
  
        this.rearrangeTodos();
  
        this.cache.currentContent = '';
      },
      /**
       * @param {Number} id - ID property of the todo
       * @description This method is used to delete the related todo when the trash icon is pressed.
       */
      removeTodo(id) {
        [this.cache.lastDeletedTodo] = this.data.todos.splice(
          this.data.todos.findIndex(todo => {
            return todo.id === id;
          }),
          1
        );
  
  
        this.rearrangeTodos();
      },
  
      /**
       * @param {object} todoObj - Todo object in todos array or this.cache.lastDeletedTodo
       * @description This method is a handler written to avoid duplication of the same code block. It is used in both addTodo and recoverTodo methods.
       */
      unshiftOrPushHandler(todoObj) {
            this.data.preferences.sortDesc
              ? this.data.todos.unshift(todoObj)
              : this.data.todos.push(todoObj);
        // }
      },
  
      /**
       * @description - This method is a handler for sorting todos by date/name or ascending/descending.
       */
      rearrangeTodos() {
            this.data.preferences.sortDesc
              ? this.data.todos.sort((a, b) => parseFloat(b.date) - parseFloat(a.date))
              : this.data.todos.sort((a, b) => parseFloat(a.date) - parseFloat(b.date));
            this.cache.filteredTodos = this.data.todos;
      },
      async loadNextImage()
              {
                  try{
                      axios.defaults.headers.common['x-api-key'] = "live_iRgQ6w9t2vbK7bmxnAaX97y4dQXBVPPFVhAGBtIbL3knFHFQDLJ5wRFajABSHFKn" // Replace this with your API Key
  
                      let response = await axios.get('https://api.thecatapi.com/v1/images/search', { params: { limit:1, size:"full" } } ) // Ask for 1 Image, at full resolution
                      
                      this.image = response.data[0] // the response is an Array, so just use the first item as the Image
  
                      this.data.imageUrl = this.image.url;
                      console.log("-- Image from TheCatAPI.com")
                      console.log("id:", this.image.id)
                      console.log("url:", this.image.url)
  
                  }catch(err){
                      console.log(err)
                  }
              },
    },
    components: {
      CheckSquare,
      Square,
      Pencil,
      Trash,
      PlusCircle,
      SwitchVertical,
      Circle,
      RecordCircle,
      ChevronRight,
      ChevronBottom,
    },
  };
  </script>
  
  <template>
    <div class="p-6 md:p-12 gap-6 md:gap-12 flex flex-col">
      <div class="w-full flex flex-col gap-3">
        <div>
          <h1 class="text-4xl font-bold dark:text-gray-200 text-center">
            To-Do List
          </h1>
        </div>
        <p v-if="data.todos.length === 0" class="dark:text-gray-300">
          
        </p>
        <div class="w-full flex flex-col gap-3">
          <div
            class="w-full flex gap-3 md:gap-6"
            v-for="todo in filterTodos"
            :key="todo.id"
          >
            <div
              class="w-1/12 flex justify-center"
              :class="
                data.preferences.date !== 'hide' && data.preferences.date !== undefined
                  ? 'mt-2'
                  : 'mt-0'
              "
            >
              <a
                href="#"
                @click.prevent
                v-if="todo.completed"
                @click="toggleStatus(todo.id)"
                class="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300 transition-all duration-200"
              >
                <CheckSquare />
              </a>
              <a
                href="#"
                @click.prevent
                v-else
                @click="toggleStatus(todo.id)"
                class="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300 transition-all duration-200"
              >
                <Square />
              </a>
            </div>
            <div class="w-10/12 flex flex-col gap-1/2">
              <input
                type="text"
                :name="todo.id"
                :id="todo.id"
                v-model="todo.content"
                class="w-full dark:bg-gray-800 dark:text-gray-300"
                :class="
                  todo.completed ? 'line-through text-gray-400 dark:text-gray-600' : ''
                "
              />
            </div>
            <div
              class="w-1/12 flex gap-3 justify-center"
              :class="
                data.preferences.date !== 'hide' && data.preferences.date !== undefined
                  ? 'mt-2'
                  : 'mt-0'
              "
            >
              <a
                href="#"
                @click.prevent
                @click="removeTodo(todo.id)"
                class="text-gray-700 dark:text-gray-200 hover:text-red-500 dark:hover:text-red-400 transition-all duration-200"
              >
                <Trash />
              </a>
            </div>
          </div>
        </div>
  
        <div class="w-full flex gap-3 md:gap-6">
          <div class="w-1/12 flex justify-center"></div>
          <div class="w-10/12">
            <input
              type="text"
              name="todoContent"
              id="todoContent"
              class="rounded w-full bg-gray-100 dark:bg-gray-700 dark:text-white px-3"
              v-model="cache.currentContent"
              v-on:keyup.enter="addTodo"
              placeholder="Enter new item and click add button"
            />
          </div>
          <div class="w-1/12 flex gap-3 justify-center">
            <a
              href="#"
              @click.prevent
              @click="addTodo"
              class="text-gray-700 dark:text-gray-200 hover:text-green-700 dark:hover:text-green-200 transition-all duration-200"
            >
              <PlusCircle />
            </a>
          </div>
        </div>
  
        <div class="relative flex py-5 items-center mt-[200px]">
          <div class="flex-grow border-t border-gray-400"></div>
          <div class="flex-grow border-t border-gray-400"></div>
        </div>
  
        <div class=" flex justify-center ">
          <div class=" justify-center">
            <button class="btn btn-blue w-full flex justify-center" @click="loadNextImage">
              Hit Me!
            </button>
            
          </div>
          
        </div>
        <div class="w-full flex justify-center  mt-[50px]">
              <img  v-bind:src=data.imageUrl /> 
            </div>
  
      </div>
  
    </div>
   
    
  </template>
  
  <style scoped>
    .btn {
      @apply font-bold py-2 px-4 rounded;
    }
    .btn-blue {
      @apply bg-blue-500 text-white;
    }
    .btn-blue:hover {
      @apply bg-blue-700;
    }
  </style>
  