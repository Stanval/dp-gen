<style lang="scss">
@import 'sass/config';
@import 'sass/reset';
@import 'sass/base';

.app {
  @extend %row;
  min-height: 100%;
}

.template {
  width: 300px;
  height: 300px;
  padding: 10px;
}

.controls {
  @extend %padding30;
  min-height: 100%;
  width: 300px;
  min-width: 300px;
  background-color: $accent-color;
  border-right: 10px solid $dark-color;
}

.wrapper {
  position: fixed;
  width: 230px;
  &>* {
    margin-bottom: 15px;
  }
}

.input-file {
  width: 0.1px;
  height: 0.1px;
  opacity: 0;
  overflow: hidden;
  position: absolute;
  z-index: -1;
}

.choose-file {
  @extend %row;
  height: 30px;
  justify-content: center;
  align-items: center;
  background-color: $accent-alt-color;
  border: 1px solid $dark-color;
  cursor: pointer;
  &:hover {
    background-color: $light-color;
  }
}

.checkbox-wrapper {
  @extend %row;
  justify-content: flex-start;
  align-items: center;
  color: $dark-color;
}

.checkbox {
  display: inline-block;
  width: 20px;
  height: 20px;
  margin-right: 15px;
}

.content {
  @extend %padding30;
  flex-grow: 1;
}
</style>

<template>
  <div id="app"
       class="app">
  
    <section class="controls">
      <div class="wrapper">
        <input type="file"
               id="file"
               class="input-file"
               @change="readFile">
        <label for="file"
               class="choose-file">Choose a file</label>
        <input type="text"
               v-model="prefix" placeholder="Dialpeer prefix">
        <input type="text"
               v-model="groups" placeholder="Allowed groups">
        <input type="text"
               v-model="rowLength" placeholder="Row length (60 default)" pattern="[0-9]*">
        <div class="checkbox-wrapper">
          <input type="checkbox"
                 id="isSpecial"
                 class="checkbox"
                 v-model="isSpecial">
          <label for="isSpecial">Is special</label>
        </div>
        <div class="checkbox-wrapper">
          <input type="checkbox"
                 id="isComplete"
                 class="checkbox"
                 v-model="isComplete">
          <label for="isComplete">Is complete</label>
        </div>
      </div>
    </section>
    <section class="content">
      <pre>{{cfg}}</pre>
    </section>
  
  </div>
</template>

<script>
export default {
  name: 'app',
  data() {
    return {
      msg: [],
      isSpecial: false,
      isComplete: false,
      rowLength: 60,
      prefix: "",
      groups: ""
    }

  },
  methods: {
    readFile(event) {
      let file = event.target.files[0],
        vue = this,
        reader = new FileReader();
      reader.onload = () => vue.msg = this.rearrange(reader.result);
      reader.readAsText(file);

    },

    rearrange(text) {
      return text
        .trim("\r\n")
        .split("\r\n")
        .map(item => item.split(";"))
        .reduce((result, [code, dst, date, route]) => {
          let priority = this.prioritize(code);
          let el = result
            .filter(e =>
              e.dst === dst
              && e.date === date
              && e.route === route
              && e.priority === priority)[0];
          if (el) el.codes.push(code)
          else result.push({
            dst: dst,
            date: date,
            route: route,
            priority: priority,
            codes: [code]
          });
          return result;
        }, []);
    },

    prioritize(code) {
      return code.length * 100 +
        (this.isSpecial ? 10 : 0) +
        (this.isComplete ? 2000 : 0);
    }
  },

  computed: {
    cfg() {
      return this.msg.map(dp =>
        '[' + this.prefix + dp.dst + ']'
        + '\n' + 'priority=' + this.prioritize(dp.codes[0], this.isSpecial, this.isComplete)
        + '\n' + 'gateway=' + dp.route
        + '\n' + divide(dp.codes, this.rowLength).map(c => 'dst_pattern=^(' + c + ').*;').join('\n')
        + '\n' + 'group_allow=' + this.groups
        + '\n' + 'hunt_stop=1')
        .join('\n\n');
    }
  }
};

function divide(set, limit) {
  return set.reduce(({ result = [[]], i = 0 }, e) => {
    [result[i], e].join('|').length <= limit ? result[i].push(e) : result[++i] = [e];
    return { 'result': result, 'i': i };
  }, {})
    .result.map(e => e.join('|'));
}

</script>

