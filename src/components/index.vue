<template>
<div>
  <input type="checkbox" id="scroll" v-model="staticPaging"/>
  <label for="scroll">Пагинация</label>
      <oz-table v-if="staticPaging"
        :rows="rows"
        :total-pages="100"
        :current-page="currentPage"
        :static-paging="staticPaging"

        @getPage="getData"
        @applySort="applySort"
        @applyFilter="applyFilter"
      >
        <oz-table-column prop="id" title="ID" />
        <oz-table-column prop="status" title="Status" />
        <oz-table-column prop="name" title="Name" />
      </oz-table>

       <oz-table v-else
        :rows="rows"
        :total-pages="100"
        :current-page="currentPage"
        :static-paging="false"

        @getPage="infGetPage"
        @applySort="applySort"
        @applyFilter="applyFilter"
      >
        <oz-table-column prop="id" title="ID" />
        <oz-table-column prop="status" title="Status" />
        <oz-table-column prop="name" title="Name" />
      </oz-table>
  </div>
</template>

<script>
import OzTable from './oz-table';
import OzTableColumn from './oz-table-column';

export default {
  name: 'PagingWrapper',
  components: {
    OzTableColumn,
    OzTable
  },
  mounted() {
    this.blockingPromise = this.getPage(1);
  },
  data() {
    return {
      rows: [],
      currentPage: 1,
      sortParam: '',
      filterParam: '',
      staticPaging: false,
      arrayOfRowsIds: []
    };
  },
  computed: {
    getData () {
      return this.staticPaging ? this.getPage : this.infGetPage
    }
  },
  watch: {
    staticPaging(value) {
      this.rows = [];
      this.arrayOfRowsIds = [];
      if (value) {
        this.currentPage = 1;
      } else {
        this.currentPage = 0;
      }
      
      if (value) {
        this.getPage(this.currentPage);
      } else {
        this.infGetPage(this.currentPage)
      }
    }
  },
  methods: {
    async getPage(pageNumber) {
      try {
        const urlQuery = [];
        console.log(this.filterParam);
        if (this.filterParam) {
          urlQuery.push(`search=${this.filterParam}`)
        }
        urlQuery.push(`limit=5`)
        urlQuery.push(`page=${pageNumber}`)
        
        if (this.sortParam) {
          urlQuery.push(`order=${this.sortParam}`)
        }

        const URL = `https://shikimori.one/api/animes?${urlQuery.join('&')}`;
        const res = await fetch(URL);
        const rows = await res.json();

        this.rows = rows
        this.currentPage = pageNumber
      } catch (err) {
        console.error(err)
      }
    },
    async infGetPage() {
      // this.blockingPromise && await this.blockingPromise;
      // const res = await fetch(`https://shikimori.one/api/animes?limit=5`);
      // const newRows = await res.json();
      // this.rows = [...this.rows, ...newRows];
      // this.currentPage++;
      this.currentPage++
      this.blockingPromise;
      try {
        const urlQuery = [];
        urlQuery.push(`page=${this.currentPage}`)
        urlQuery.push(`limit=5`)
        
        if (this.sortParam) {
          urlQuery.push(`order=${this.sortParam}`)
        }

        const URL = `https://shikimori.one/api/animes?${urlQuery.join('&')}`;
        const res = await fetch(URL);
        const rows = await res.json();

        const newRows = rows.reduce((acc, element) => {
          if (!this.arrayOfRowsIds.includes(element.id)) {
            return {
              elements: [...acc.elements, element],
              ids: [...acc.ids, element.id]
            }
          } 
          return acc
        }, { elements: [], ids: [] });
        
        console.log(newRows);
        if (newRows) {
          this.rows = [...this.rows, ...newRows.elements];
          this.arrayOfRowsIds= [...this.arrayOfRowsIds, ...newRows.ids];
        }
        
      } catch (err) {
        console.error(err)
      }
    },

    applySort(prop) {
      this.filterParam = '';
      if (this.sortParam !== prop) {
        this.sortParam = prop;
      } else {
        this.sortParam = ''
      }
      if (this.staticPaging) {
      this.currentPage = 1;
      } else {
        this.currentPage = 0;
      }
      this.arrayOfRowsIds = [];
      this.rows = []
      if (this.staticPaging) {
        this.getPage(this.currentPage)
      } else {
        this.infGetPage()
      }
    },
    applyFilter(prop) {
      this.currentPage = 1;
      this.filterParam = prop;
      console.log(this.filterParam);
      this.getPage(this.currentPage)
    }
  }
};
</script>
