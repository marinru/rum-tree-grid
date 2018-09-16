<template>
  <div id="app">
    <TreeGrid :cols="cols">
      <div slot="id-heading" class="cell-heading">id</div>
      <div slot="name-heading" class="cell-heading">Имя</div>
      <div slot="age-heading" class="cell-heading">Возраст</div>
      <TreeGridBody :cols="cols" :childProps="childProps" :treeData="treeData">
        <div slot="id"
          slot-scope="{cellData, isParent, isOpen, branch}"
          :class="['cell cell-0', {'cell-parent': isParent}]"
          >
          <div v-if="isParent">
            <span class="icon" v-text="isOpen ? 'v' : '>'" @click="$emit('clicked')"></span>
          </div>
          <div v-if="branch" v-text="branch"></div>
          <div v-else v-text="cellData"></div>
        </div>
        <div slot="name"
          slot-scope="{cellData, isParent, branch}"
          :class="['cell', {'cell-parent': isParent}]"
          v-text="branch ? '' : cellData">
        </div>
        <div slot="age"
          slot-scope="{cellData, isParent, branch}"
          :class="['cell', {'cell-parent': isParent}]"
          v-text="branch ? '' : cellData">
        </div>
      </TreeGridBody>      
    </TreeGrid>
  </div>
</template>

<script>
import TreeGrid from "./components/TreeGrid";
import TreeGridBody from "./components/TreeGridBody";
import seed from "./seed.json";

export default {
  name: "App",
  components: { TreeGrid, TreeGridBody },
  data() {
    return {
      treeData: seed,
      cols: ["id", "name", "age"],
      childProps: nodeData => [
        { name: "children", immediate: true, loader: (nodeData) => {
          return Promise.resolve([
            { id: 20000, name: "Jack", age: 7 },
            { id: 20002, name: "Leo", age: 5 }
          ]);
        }},
        { name: "friends" }
      ]      
    };
  }
};
</script>

<style>
.cell-heading {
  width: 100%;
  height: 100%;
  padding: 20px;
  background-color: #f1f4f7;
  box-sizing: border-box;
  display: flex;
  justify-content: center;
  align-items: center;
}

.cell {
  display: flex;
  flex: 1;
  justify-content: center;
  align-items: center;
  background: white;
  
}

.cell-0 {
  justify-content: flex-start;
  padding-left: 5px;
}

.cell-parent {
  background: #e2e2e2;
}

.icon {
  display: inline-block;
  margin: 0 5px;
  line-height: 16px;
  font-family: sans-serif;
  height: 16px;
  width: 16px;
  background-color: #5fcfff;
  vertical-align: middle;
  text-align: center;
  user-select: none;
  cursor: pointer;
}

</style>
