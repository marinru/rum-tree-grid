<script>
export default {
  props: { cols: Array, childProps: [Array, Function], treeData: Array },
  data() {
    return { 
    };
  },
  methods: {

    getChildProps(nodeData) {
      let childProps = typeof(this.childProps) === "function"
        ? this.childProps(nodeData)
        : this.childProps;

      return childProps;
    },

    toggleOpen(nodeData, branch) {
      let isStubBranch = branch && !nodeData[branch];
      if (isStubBranch) {
        return;
      }
      
      let target = branch ? nodeData[branch] : nodeData;
      this.$set(target, "_isOpen", !target._isOpen);
    },

    getCellWrapperClass(isParent, cellIndex) {
      return ["tree-grid__cell-wrapper", { "parent": isParent, "col-0": cellIndex === 0 }];
    },

    getCellWrapperStyle(cellIndex, level){
      return {
        "padding-left": cellIndex === 0 ? `${level * 20}px` : ""
        };
    },

    renderCellContent(h, nodeData, level, isParent, isOpen, branch, col, cellIndex) {
      return h(
        "div",
          {
            class: this.getCellWrapperClass(isParent, cellIndex),
            style: this.getCellWrapperStyle(cellIndex, level)
          },
        [
          this.$scopedSlots[col]({cellData: nodeData[col], isParent, isOpen, branch})
        ]
      );
    },

    renderCells(h, nodeData, level, isParent, isOpen, branch) {
      return this.cols.map((col, cellIndex) => h("td", { class: "tree-grid__cell" }, [
        this.renderCellContent(h, nodeData, level, isParent, isOpen, branch, col, cellIndex)
      ]));
    },

    renderChildBranch(h, nodeData, childBranchProp, level) {
      let isOpen = nodeData[childBranchProp.name]._isOpen || childBranchProp.immediate;
      return [
        h("tr",
          {
            class: "tree-grid__row",
            on: {
              click: e => {
                // get data for this branch
                this.toggleOpen(nodeData, childBranchProp.name);
              }
            }
          },
          [childBranchProp.immediate
            ? []
            : this.renderCells(h, nodeData, level, true, isOpen, childBranchProp.name)
          ]
        ),
        ...(isOpen ? (nodeData[childBranchProp.name] || []).map(childNodeData =>
          this.renderBranch(h, childNodeData, childBranchProp.immediate ? level :level + 1)) : [])
      ];
    },

    renderBranch(h, nodeData, level, branch) {
      //console.log("renderNodes");
      let childProps = this.getChildProps(nodeData);
      let childPropNames = childProps.map(cp => cp.name);
      let isParent = (childPropNames || []).some(p => Object.keys(nodeData).includes(p));
      let isOpen = isParent && nodeData._isOpen;
      return [
        // Render the root node
        h("tr",
          {
            class: "tree-grid__row",
            on: {
              click: e => {
                if (!nodeData._isOpen && !nodeData._isLoaded) {
                  let childBranchesToLoad =
                    childProps.filter(cp => cp.immediate
                      && cp.loader
                      && Object.keys(nodeData).includes(cp.name));
                  
                  let loaderPromises =
                    childBranchesToLoad.map(c => c.loader(nodeData).then(data => nodeData[c.name] = (data || [])));

                  Promise.all(loaderPromises).then(() => {
                    this.$set(nodeData, "_isLoaded", true);
                    this.toggleOpen(nodeData);
                    });

                  } else {
                    this.toggleOpen(nodeData);
                  }                
              }
            }
          },
          [this.renderCells(h, nodeData, level, isParent, isOpen)]),
        // Render child nodes
        ...(isOpen ? childProps
          .filter(childProp => Object.keys(nodeData).includes(childProp.name))
          .map(childProp =>
            this.renderChildBranch(h, nodeData, childProp, level + 1)) : [])
      ];
    }
  },  

  render(h) {
    return h("tbody", {}, [ this.treeData.map(root => this.renderBranch(h, root, 0)) ]);
  }
};
</script>

<style>
.tree-grid__cell-wrapper {
    display: flex;
    justify-content: center;
    align-items: stretch;
    height: 34px;
}

.tree-grid__cell {
  padding: 0;
  border-left: 1px dashed #e2e2e2;
  background: #f5f5f5;
}

.tree-grid__cell:first-child {
    border-left: none;
}

.parent {
  font-weight: bold;
}

.col-0 {  
  justify-content: left;
}
</style>
