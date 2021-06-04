Vue 3 drag-and-drop component based on Sortable.js.

```
npm install vue-draggable-next
//or
yarn add vue-draggable-next
```

```
<template>
    <div class="sort-table-js" > 
        <h3>Draggable {{ draggingInfo }}</h3>
        <draggable
        :persons="persons"
        :disabled="!enabled"
        item-key="name"
        class="list-group"
        ghost-class="ghost"
        v-bind="dragOptions"
        :move="checkMove"
        @start="dragging = true"
        @end="dragging = false"
      >
      <transition-group type="transition" name="flip-list">
       <el-card  class="box-card" v-for="element in persons"  :key="element.name">
            <div class="text item list-group-item"  :class="{ 'not-draggable': !enabled }">
                <h3>{{element .name}}</h3>
                <div>{{element .description}}</div>
            </div>
       </el-card>
      </transition-group>
    </draggable>
    </div>
</template>
<script>
  import { defineComponent } from 'vue'
  import { VueDraggableNext } from 'vue-draggable-next'
export default defineComponent({
    name:'SortTableJs',
    components: {draggable : VueDraggableNext},
    data() {
        return {
            dragging: false,
            enabled: true,
            persons: [
                {
                    id:0,
                    name: "Elliot Fu",
                    description: "Elliot Fu is a film-maker from New York."
                },
                 {
                    id:1,
                    name: "Veronika Ossi",
                    description: "Input array to draggable component. Typically same array as referenced by inner element v-for directive."
                },
                 {
                    id:2,
                    name: "Jenny Hess",
                    description: "Veronika Ossi is a set designer living in New York who enjoys kittens, music, and partying."
                },
                 {
                    id:3,
                    name: "Jacky Chan",
                    description: "enny is a student studying Media Management at the New School."
                },
            ]
        }
    },
computed: {
    draggingInfo() {
      return this.dragging ? "under drag" : "";
    },
    dragOptions() {
      return {
        animation: 0,
        group: 'description',
        disabled: false,
        ghostClass: 'ghost',
      }
    }
  },
    methods: {
         checkMove: function(e) {
            window.console.log("Future index: " + e.draggedContext.futureIndex);
        }
    }
})
</script>
<style scoped>
.box-card {
    margin:10px 0px;
    cursor: pointer;
}
.buttons {
  margin-top: 35px;
}
.ghost {
  opacity: 0.5;
  background: #c8ebfb;
}
.not-draggable {
  cursor: no-drop;
}
</style>
```

https://vuejsexamples.com/a-simple-vue-3-drag-and-drop-component/

