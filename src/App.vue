<script setup>
import { ref } from 'vue';
import { Chrome } from 'vue-color';

// 定义待办事项列表
const todos = ref([
  
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

// 拖曳排序相关变量
const draggedTodo = ref(null);
const dragStartY = ref(0);
const dragThreshold = 15; // 垂直拖动阈值，单位像素
const isDragging = ref(false);
const dragOverTodo = ref(null);

// 开始拖动
const startDrag = (event, todo) => {
  // 如果正在编辑，不允许拖动
  if (editingTodo.value) return;
  
  // 阻止默认行为，防止文本选择
  event.preventDefault();
  
  draggedTodo.value = todo;
  dragStartY.value = event.clientY;
  
  // 添加全局鼠标事件监听
  document.addEventListener('mousemove', onDrag);
  document.addEventListener('mouseup', endDrag);
};

// 拖动过程
const onDrag = (event) => {
  if (!draggedTodo.value) return;
  
  // 计算垂直移动距离
  const deltaY = event.clientY - dragStartY.value;
  
  // 只有垂直移动超过阈值才触发拖动效果
  if (Math.abs(deltaY) >= dragThreshold) {
    isDragging.value = true;
    
    // 获取鼠标下方的元素
    const elementsUnderMouse = document.elementsFromPoint(event.clientX, event.clientY);
    
    // 查找鼠标下方的待办事项
    for (const element of elementsUnderMouse) {
      const todoItem = element.closest('.todo-item');
      if (todoItem) {
        const todoId = parseInt(todoItem.getAttribute('data-id'));
        const overTodo = todos.value.find(t => t.id === todoId && !t.completed);
        
        if (overTodo && overTodo !== draggedTodo.value) {
          dragOverTodo.value = overTodo;
          return;
        }
      }
    }
    
    dragOverTodo.value = null;
  }
};

// 结束拖动
const endDrag = () => {
  if (isDragging.value && draggedTodo.value && dragOverTodo.value) {
    // 获取源和目标索引
    const activeTodos = todos.value.filter(t => !t.completed);
    const sourceIndex = activeTodos.findIndex(t => t.id === draggedTodo.value.id);
    const targetIndex = activeTodos.findIndex(t => t.id === dragOverTodo.value.id);
    
    if (sourceIndex !== -1 && targetIndex !== -1) {
      // 从数组中移除拖动项
      const [movedItem] = activeTodos.splice(sourceIndex, 1);
      // 插入到新位置
      activeTodos.splice(targetIndex, 0, movedItem);
      
      // 更新原始数组
      const newTodos = [];
      // 先添加未完成的项（已排序）
      newTodos.push(...activeTodos);
      // 再添加已完成的项
      newTodos.push(...todos.value.filter(t => t.completed));
      todos.value = newTodos;
    }
  }
  
  // 重置拖动状态
  draggedTodo.value = null;
  dragOverTodo.value = null;
  isDragging.value = false;
  
  // 移除全局事件监听
  document.removeEventListener('mousemove', onDrag);
  document.removeEventListener('mouseup', endDrag);
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
        <li 
          v-for="todo in todos.filter(t => !t.completed)" 
          :key="todo.id" 
          class="todo-item" 
          :class="{
            'dragging': isDragging && draggedTodo === todo,
            'drag-over': dragOverTodo === todo
          }"
          :data-id="todo.id"
          @mousedown="startDrag($event, todo)"
        >
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
  cursor: pointer;
  position: relative;
  transition: transform 0.2s, box-shadow 0.2s, background-color 0.2s;
}

.todo-item.dragging {
  opacity: 0.7;
  background-color: #2a2a2a;
  box-shadow: 0 0 8px rgba(0, 0, 0, 0.5);
  z-index: 10;
  user-select: none;
}

.todo-item.drag-over {
  border-top: 2px solid #42b883;
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
  margin-top: 15px;
  font-size: 1rem;
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

.standard-colors {
  display: flex;
  flex-wrap: wrap;
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
