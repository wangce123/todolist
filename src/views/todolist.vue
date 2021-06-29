<template>
  <div class="todolist">
    <textarea class="task-ipt" cols="30" rows="10" v-model="text"></textarea>
    <button class="btn-add" @click="myadd">添加</button>
    <ul class="tast-list">
      <li class="tast-item" v-for="(item, index) in tastlist" :key="index">
        {{ item.text }}{{ item.num }}
        <button class="btn-delete" @click="mydelete(index)">删除</button>
      </li>
    </ul>
  </div>
</template>
 
<script>
export default {
  name: "Todolist",
  data() {
    return {
      text: "",
      tastlist: [
        {
          text: "aa",
          num: "2",
        },
      ],
    };
  },
  methods: {
    myadd: function () {
      let obj = {
        text: this.text,
        num: this.text.length,
      };
      this.tastlist.push(obj);
    },
    mydelete: function (index) {
      this.tastlist.splice(index, 1);
    },
  },
};
</script>
 
<style scoped lang = "scss">
.todolist {
  .task-ipt {
    width: 750px;
    height: 80px;
    border: 1px solid red;
    margin: 10px auto;
    display: block;
  }
  .btn-add {
    margin: 5px auto;
  }
  .tast-list {
    list-style: none;
    width: 800px;
    margin: 10px auto;
    display: flex;
    .tast-item {
      width: 400px;
      height: 100px;
      display: block;
      border: 1px solid red;
    }
  }
}
</style>