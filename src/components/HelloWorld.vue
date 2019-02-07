<template>
  <div class="hello">
    <h1>チュートリアルToDoリスト</h1>
    <!-- ★STEP11 -->
    <label v-for="label in options">
      <input type="radio"
             v-model="current"
             v-bind:value="label.value">{{ label.label }}
    </label>
    <!-- 絞り込みラジオボタン -->
    <!-- ToDo テーブル -->
    <table>
      <!-- テーブルヘッダー -->
      <thead>
      <tr>
        <th class="id">ID</th>
        <th class="comment">コメント</th>
        <th class="state">状態</th>
        <th class="button">-</th>
      </tr>
      </thead>
      <tbody>
      <!-- ここに <tr> で ToDo の要素を1行づつ繰り返し表示 -->
      <tr v-for="item in computedTodos" v-bind:key="item.id">
        <th>{{ item.id }}</th>
        <td>{{ item.comment }}</td>
        <td class="state">
          <!-- 状態変更ボタンのモック -->
          <button v-on:click="doChangeState(item)">
            {{ labels[item.state] }}
          </button>
        </td>
        <td class="button">
          <!-- 削除ボタンのモック -->
          <button v-on:click="doRemove(item)">
            削除
          </button>
        </td>
      </tr>
      </tbody>
    </table>
    <!-- 新規登録フォーム -->
    <h2>新しい作業の追加</h2>
    <form class="add-form" v-on:submit.prevent="doAdd">
      <!-- コメント入力フォーム -->
      コメント <input type="text" ref="comment">
      <!-- 追加ボタンのモック -->
      <button type="submit">追加</button>
    </form>
  </div>
</template>

<script>
  export default {
    name: 'HelloWorld',
    data() {
      return {
        msg: 'Happy hello',
        todos: [],
        current: -1,
        options: [
          {value: -1, label: 'すべて'},
          {value: 0, label: '作業中'},
          {value: 1, label: '完了'},
        ],
      };
    },
    methods: {
      doAdd: function(event, value) {
        // ref で名前を付けておいた要素を参照
        const comment = this.$refs.comment;
        // 入力がなければ何もしないで return
        if (!comment.value.length) {
          return;
        }
        // { 新しいID, コメント, 作業状態 }
        // というオブジェクトを現在の todos リストへ push
        // 作業状態「state」はデフォルト「作業中=0」で作成
        this.todos.push({
          id: todoStorage.uid++,
          comment: comment.value,
          state: 0,
        });
        // フォーム要素を空にする
        comment.value = '';
      },
      doChangeState: function(item) {
        item.state = item.state ? 0 : 1;
      },
      doRemove: function(item) {
        const index = this.todos.indexOf(item);
        this.todos.splice(index, 1);
      },
    },
    watch: {
      // オプションを使う場合はオブジェクト形式にする
      todos: {
        // 引数はウォッチしているプロパティの変更後の値
        handler: function(todos) {
          todoStorage.save(todos);
        },
        // deep オプションでネストしているデータも監視できる
        deep: true,
      },
    },
    created() {
      // インスタンス作成時に自動的に fetch() する
      this.todos = todoStorage.fetch();
    },
    computed: {
      labels() {
        return this.options.reduce(function(a, b) {
          return Object.assign(a, {[b.value]: b.label});
        }, {});
        // キーから見つけやすいように、次のように加工したデータを作成
        // {0: '作業中', 1: '完了', -1: 'すべて'}
      },
      computedTodos: function() {
        // データ current が -1 ならすべて
        // それ以外なら current と state が一致するものだけに絞り込む
        return this.todos.filter(function(el) {
          return this.current < 0 ? true : this.current === el.state
        }, this)
      }
    }
  };
  const STORAGE_KEY = 'todos-vuejs-demo';
  const todoStorage = {
    fetch: function() {
      const todos = JSON.parse(
        localStorage.getItem(STORAGE_KEY) || '[]'
      );
      todos.forEach(function(todo, index) {
        todo.id = index;
      });
      todoStorage.uid = todos.length;
      return todos;
    },
    save: function(todos) {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
    },
  };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }
</style>
