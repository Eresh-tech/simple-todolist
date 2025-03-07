<script setup>
import { ref } from 'vue';
import { Chrome } from 'vue-color';

// 定义待办事项列表
const todos = ref([
  { id: 1, text: '3月11日', completed: false, color: '#FFFFFF', createdAt: new Date('2023-03-11T10:00:00'), completedAt: null },
  { id: 2, text: '1、翻译论文', completed: false, color: '#FFFFFF', createdAt: new Date('2023-03-12T09:30:00'), completedAt: null },
  { id: 3, text: '2、排版', completed: false, color: '#FFFFFF', createdAt: new Date('2023-03-12T14:00:00'), completedAt: null },
  { id: 4, text: '打游戏', completed: false, color: '#FFFFFF', createdAt: new Date('2023-03-13T18:00:00'), completedAt: null }
]);

// 格式化日期时间
const formatDateTime = (date) => {
  if (!date) return '';
  const d = new Date(date);
  const month = d.getMonth() + 1;
  const day = d.getDate();
  const hours = d.getHours().toString().padStart(2, '0');
  const minutes = d.getMinutes().toString().padStart(2, '0');
  return `${month}月${day}日 ${hours}:${minutes}`;
};

// 当前选中的标签页（todo 或 done）
const activeTab = ref('todo');

// 添加新的待办事项
const addTodo = () => {
  const todo = {
    id: Date.now(),
    text: '',
    completed: false,
    color: '#FFFFFF'
  };
  todos.value.push(todo);
  startEdit(todo);
};

// 切换待办事项状态
const toggleTodo = (todo) => {
  todo.completed = !todo.completed;
  // 如果标记为已完成，记录完成时间
  if (todo.completed) {
    todo.completedAt = new Date();
  } else {
    todo.completedAt = null;
  }
};

// 显示新增待办事项输入框
const showAddInput = ref(false);
const toggleAddInput = () => {
  showAddInput.value = !showAddInput.value;
};

// 删除待办事项
const deleteTodo = (id) => {
  const index = todos.value.findIndex(todo => todo.id === id);
  if (index !== -1) {
    todos.value.splice(index, 1);
  }
};

// 编辑待办事项
const editingTodo = ref(null);
const editText = ref('');
const editColor = ref('#FFFFFF');
const showColorPicker = ref(false);

// 颜色选择器预设颜色
const themeColors = [
  ['#000000', '#F5F5DC', '#0F52BA', '#6495ED', '#C41E3A', '#7CFC00', '#808000', '#40E0D0', '#FFA500'],
  ['#808080', '#D2B48C', '#ADD8E6', '#87CEEB', '#FFC0CB', '#E6E6FA', '#D8BFD8', '#AFEEEE', '#FFE4B5'],
  ['#000000', '#2F4F4F', '#696969', '#778899', '#708090', '#A9A9A9', '#C0C0C0', '#D3D3D3', '#DCDCDC'],
  ['#556B2F', '#8B4513', '#A0522D', '#6B8E23', '#BDB76B', '#F4A460', '#CD853F', '#DEB887', '#D2B48C'],
  ['#483D8B', '#4682B4', '#4169E1', '#191970', '#000080', '#00008B', '#0000CD', '#0000FF', '#1E90FF'],
  ['#8B0000', '#800000', '#A52A2A', '#B22222', '#DC143C', '#FF0000', '#FF6347', '#FF7F50', '#CD5C5C'],
  ['#9400D3', '#800080', '#8B008B', '#BA55D3', '#9932CC', '#8A2BE2', '#9370DB', '#7B68EE', '#6A5ACD'],
  ['#008080', '#008B8B', '#5F9EA0', '#20B2AA', '#00CED1', '#48D1CC', '#00FFFF', '#7FFFD4', '#66CDAA'],
  ['#FF4500', '#FF8C00', '#FFA500', '#FFD700', '#FFFF00', '#F0E68C', '#EEE8AA', '#DAA520', '#B8860B']
];

const standardColors = [
  '#FF0000', '#FF7F00', '#FFFF00', '#00FF00', '#00FFFF', '#0000FF', '#8B00FF', '#000000', '#FFFFFF'
];

const startEdit = (todo) => {
  editingTodo.value = todo;
  editText.value = todo.text;
  editColor.value = todo.color || '#FFFFFF';
  showColorPicker.value = false;
};

const saveEdit = () => {
  if (editingTodo.value && editText.value.trim()) {
    editingTodo.value.text = editText.value;
    editingTodo.value.color = editColor.value;
    editingTodo.value = null;
    showColorPicker.value = false;
  }
};

const cancelEdit = () => {
  editingTodo.value = null;
  showColorPicker.value = false;
};

const toggleColorPicker = () => {
  showColorPicker.value = !showColorPicker.value;
};

const updateColor = (color) => {
  editColor.value = color.hex;
};

const selectThemeColor = (color) => {
  editColor.value = color;
  showColorPicker.value = false;
};
</script>

<template>
  <div class="todo-container">
    <!-- 标签页切换 -->
    <div class="tabs">
      <div 
        class="tab" 
        :class="{ active: activeTab === 'todo' }" 
        @click="activeTab = 'todo'"
      >
        ToDo
      </div>
      <div 
        class="tab" 
        :class="{ active: activeTab === 'done' }" 
        @click="activeTab = 'done'"
      >
        Done
      </div>
    </div>
    
    <!-- Todo列表 -->
    <div v-if="activeTab === 'todo'" class="todo-list">
      <div v-if="todos.filter(t => !t.completed).length === 0" class="empty-state">
        <p>暂无待办事项</p>
      </div>
      <ul v-else>
        <li v-for="todo in todos.filter(t => !t.completed)" :key="todo.id" class="todo-item">
          <!-- 编辑模式 -->
          <div v-if="editingTodo === todo" class="edit-mode">
            <input 
              v-model="editText" 
              @keyup.enter="saveEdit"
              class="edit-input"
              autofocus
            />
            <div class="color-picker-container">
              <button 
                @click="toggleColorPicker" 
                class="color-btn" 
                :style="{ backgroundColor: editColor }"
              ></button>
              <div v-if="showColorPicker" class="color-picker-dropdown">
                <div class="color-picker-section">
                  <div class="section-title">主题颜色</div>
                  <div class="theme-colors">
                    <div v-for="(row, rowIndex) in themeColors" :key="'row-'+rowIndex" class="color-row">
                      <div 
                        v-for="(color, colIndex) in row" 
                        :key="'col-'+colIndex"
                        class="color-cell"
                        :style="{ backgroundColor: color }"
                        @click="selectThemeColor(color)"
                      ></div>
                    </div>
                  </div>
                </div>
                <div class="color-picker-section">
                  <div class="section-title">标准颜色</div>
                  <div class="standard-colors">
                    <div 
                      v-for="(color, index) in standardColors" 
                      :key="index"
                      class="color-cell"
                      :style="{ backgroundColor: color }"
                      @click="selectThemeColor(color)"
                    ></div>
                  </div>
                </div>
                <div class="color-picker-section">
                  <div class="section-title">更多颜色..</div>
                  <input 
                    type="color" 
                    v-model="editColor"
                    class="color-picker"
                  />
                </div>
              </div>
            </div>
            <div class="edit-actions">
              <button @click="saveEdit" class="icon-btn save-btn">
                <i class="fas fa-check"></i>
              </button>
              <button @click="cancelEdit" class="icon-btn cancel-edit-btn">
                <i class="fas fa-times"></i>
              </button>
            </div>
          </div>
          
          <!-- 查看模式 -->
          <div v-else class="view-mode">
            <span class="bullet"><i class="far fa-calendar-alt" :title="`创建时间: ${formatDateTime(todo.createdAt)}`"></i></span>
            <span class="todo-text" :style="{ color: todo.color }">{{ todo.text }}</span>
            <div class="todo-actions">
              <button @click="startEdit(todo)" class="icon-btn edit-btn" title="编辑">
                <i class="fas fa-edit"></i>
              </button>
              <button @click="toggleTodo(todo)" class="icon-btn complete-btn" title="完成">
                <i class="far fa-check-circle"></i>
              </button>
              <button @click="deleteTodo(todo.id)" class="icon-btn delete-btn" title="删除">
                <i class="fas fa-trash"></i>
              </button>
            </div>
          </div>
        </li>
      </ul>
      <!-- 移除了底部的添加按钮 -->
    </div>
    
    <!-- Done列表 -->
    <div v-if="activeTab === 'done'" class="todo-list">
      <div v-if="todos.filter(t => t.completed).length === 0" class="empty-state">
        <p>暂无已完成事项</p>
      </div>
      <ul v-else>
        <li v-for="todo in todos.filter(t => t.completed)" :key="todo.id" class="todo-item">
          <div class="view-mode">
            <span class="bullet"><i class="far fa-clock" :title="`完成时间: ${formatDateTime(todo.completedAt)}`"></i></span>
            <span class="todo-text completed" :style="{ color: todo.color }">{{ todo.text }}</span>
            <div class="todo-actions">
              <button @click="toggleTodo(todo)" class="icon-btn uncomplete-btn" title="未完成">
                <i class="fas fa-sync"></i>
              </button>
              <button @click="deleteTodo(todo.id)" class="icon-btn delete-btn" title="删除">
                <i class="fas fa-trash"></i>
              </button>
            </div>
          </div>
        </li>
      </ul>
    </div>
    
    <!-- 添加待办事项按钮 -->
    <button v-if="activeTab === 'todo'" @click="addTodo" class="add-new-btn" title="新增待办事项">
      <i class="fas fa-plus"></i>
    </button>
  </div>
</template>

<style>
:root {
  font-family: system-ui, Avenir, Helvetica, Arial, sans-serif;
  line-height: 1.5;
  font-weight: 400;
  color-scheme: light dark;
  color: rgba(255, 255, 255, 0.87);
  background-color: #242424;
}

body {
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  min-width: 320px;
  min-height: 100vh;
}

.todo-container {
  width: 300px;
  background-color: #1a1a1a;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.tabs {
  display: flex;
  margin-bottom: 20px;
  border-bottom: 1px solid #333;
}

.tab {
  padding: 10px 15px;
  cursor: pointer;
  font-size: 1.2rem;
  font-weight: bold;
  color: #888;
}

.tab.active {
  color: white;
}

.todo-list ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.todo-item {
  padding: 10px 0;
  border-bottom: 1px solid #333;
}

.view-mode, .edit-mode {
  display: flex;
  align-items: center;
  width: 100%;
}

.bullet {
  margin-right: 10px;
  font-size: 1.2rem;
}

.todo-text {
  font-size: 1rem;
  flex-grow: 1;
  color: white;
}

.todo-text.completed {
  text-decoration: line-through;
  color: #888;
}

.todo-actions, .edit-actions {
  display: flex;
  gap: 8px;
}

.icon-btn {
  background: none;
  border: none;
  cursor: pointer;
  color: #888;
  font-size: 1rem;
  padding: 2px;
  transition: color 0.2s;
}

.complete-btn:hover {
  color: #42b883;
}

.uncomplete-btn {
  color: #42b883;
}

.uncomplete-btn:hover {
  color: #888;
}

.edit-btn:hover {
  color: #4a9eff;
}

.delete-btn:hover {
  color: #ff4a4a;
}

.save-btn:hover {
  color: #42b883;
}

.cancel-edit-btn:hover {
  color: #ff4a4a;
}

.empty-state {
  text-align: center;
  padding: 20px 0;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.add-new-btn {
  background-color: #42b883;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
  font-size: 1rem;
}

.add-btn {
  background-color: #42b883;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.add-input-container {
  margin-top: 20px;
}

.add-input, .edit-input {
  width: 100%;
  padding: 8px;
  border: 1px solid #333;
  background-color: #2c2c2c;
  color: white;
  border-radius: 4px;
  margin-bottom: 10px;
  box-sizing: border-box;
}

.edit-input {
  margin-right: 2px;
  margin-bottom: 0;
  width: calc(100% - 74px);
}

.color-picker-container {
  position: relative;
  margin-right: 10px;
}

.color-btn {
  width: 16px;
  height: 16px;
  padding: 2px;
  border: 1px solid #333;
  cursor: pointer;
  border-radius: 4px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1rem;
}

.color-picker-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  z-index: 10;
  background-color: #2c2c2c;
  border: 1px solid #444;
  border-radius: 4px;
  padding: 10px;
  width: 220px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
  margin-top: 5px;
}

.color-picker-section {
  margin-bottom: 10px;
}

.section-title {
  font-size: 0.9rem;
  color: #aaa;
  margin-bottom: 1px;
}

.theme-colors, .standard-colors {
  display: flex;
  flex-wrap: wrap;
}

.color-row {
  display: flex;
  width: 100%;
  margin-bottom: 2px;
}

.color-cell {
  width: 20px;
  height: 20px;
  margin: 1px;
  cursor: pointer;
  border: 1px solid #444;
}

.color-cell:hover {
  border-color: #fff;
}

.color-picker {
  width: 30px;
  height: 30px;
  padding: 0;
  border: none;
  background: none;
  cursor: pointer;
  border-radius: 4px;
  overflow: hidden;
}

.add-actions {
  display: flex;
  justify-content: space-between;
}

.confirm-btn {
  background-color: #42b883;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 5px;
}

.cancel-btn {
  background-color: #666;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 5px;
}
</style>
