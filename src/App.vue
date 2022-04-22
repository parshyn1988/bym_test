<template>
  <div id="app">
    <template v-if="loading">
      <h3>Loading ...</h3>
    </template>
    <template v-else-if="handleError">
      <h3>{{ handleError }}</h3>
    </template>
    <template v-else>
      <button @click="downloadTable">Download</button>
      <table>
        <tr>
          <th v-for="headerItem in tableHeader">{{ headerItem }}</th>
        </tr>
        <tr v-for="productRow in productList">
          <td v-for="productItem in productRow">{{ productItem }}</td>
        </tr>
      </table>
    </template>
  </div>
</template>

<script>
const URL = 'https://raw.githubusercontent.com/OlegFomishyn/test-data/main/products.json',
      CUSTOM_ERROR = 'Something was wrong...';

export default {
  name: 'App',

  data() {
    return {
      handleError: '',
      loading: true,
      productList: [],
    }
  },

  created() {
    this.getData();
  },

  computed: {
    tableHeader() {
      if (this.productList.length) {
        return Object.keys(this.productList[0]);
      }
    },
  },

  methods: {
    getData() {
      fetch(URL)
          .then(response => {
            this.checkResponse(response);
            return response.json();
          })
          .then(data => {
            this.convertData(data);
          })
          .catch(error => {
            this.handleError = CUSTOM_ERROR;
            console.error(error);
          })
          .finally(() => {
            this.loading = !this.loading;
          })
    },

    checkResponse(res) {
      if (!res.ok) {
        this.handleError = CUSTOM_ERROR;
      }
    },

    convertData(arr) {
      arr.forEach((item) => {
        const tmpItem = {...item};

        tmpItem.price = `${tmpItem.price / 100}$`;
        tmpItem.included.length ? tmpItem.included = 'Yes' : tmpItem.included = 'No';

        for (let key in tmpItem) {
          tmpItem[this.convertObjectKey(key)] = tmpItem[key];
          delete tmpItem[key];
        }

        this.productList.push(tmpItem);

        if (item.included.length) {
          this.convertData(item.included);
        }
      });
    },

    convertObjectKey(value) {
      if (value === 'included') {
        value = 'is package';
      }
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    },

    downloadTable() {
      const fileName = new Date();
      const dataStr = 'data:text/json;charset=utf-8,' + encodeURIComponent(JSON.stringify(this.productList));
      const downloadAnchorNode = document.createElement('a');
      downloadAnchorNode.setAttribute('href', dataStr);
      downloadAnchorNode.setAttribute('download', fileName.toDateString() + '.json');
      downloadAnchorNode.click();
      downloadAnchorNode.remove();
    },
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 20px;
}

button {
  margin-bottom: 20px;
}

table {
  width: 100%;
  min-width: 900px;

  tr {
    text-align: left;
    height: 30px;
  }

  tr th:nth-child(1) {
    width: 10%;
  }

  tr th:nth-child(2) {
    width: 65%;
  }

  tr th:nth-child(3) {
    width: 10%;
  }

  tr th:nth-child(4) {
    width: 15%;
  }
}
</style>
