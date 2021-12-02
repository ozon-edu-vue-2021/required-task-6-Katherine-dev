<script lang="jsx">
import OzTablePaginator from './oz-table-paginator';
import DotsLoaderIcon from './dost-loader.svg';
import FilterDropdown from './filter-dropdown';

export default {
  name: 'oz-table',
  props: {
    rows: {
      type: Array,
      default: () => []
    },
    totalPages: {
      type: Number,
      default: 0
    },
    currentPage: {
      type: Number,
      default: 0
    },
    staticPaging: {
      type: Boolean,
      default: true
    }
  },
  watch: {
    staticPaging()
    {
      this.sortDirection = ''
    },
    sortProp(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.sortDirection = 'asc'
      }
    },
    filterText() {
      this.sortDirection = ''
      this.sortProp = ''
    }
  },
   data() {
    return {
      sortProp: '',
      sortDirection: '',
      filterProp: '',
      filterText: ''
    };
  },
  methods: {
    toggleSort(prop) {
      this.sortProp = prop;
      this.sortDirection = (this.sortDirection === 'none' || !this.sortDirection) ? 'asc' : 'none';
      this.$emit("applySort", prop)
      this.filterText = '';
    },
    openFilterTooltip(prop = '') {
      this.filterProp = prop;
    },
    setFilterText(e) {
      this.filterText = e.target.value;
      this.sortDirection = 'asc'
      this.$emit("applyFilter", e.target.value)
    },
    renderHead(h, columnsOptions) {
      const { $style, sortProp, sortDirection, filterProp, filterText } = this;

      return columnsOptions.map((column) => {
        const renderedTitle = column.scopedSlots.title ? column.scopedSlots.title() : column.title;
        let sortIcon = 'sort-amount-down-alt';

        if (sortProp === column.prop) {
          sortIcon = sortDirection === 'asc' ? 'times' : 'sort-amount-down-alt';
        }

        return (
          <th key={column.prop} class={$style.headerCell} data-key={column.prop}>
            <div class={$style.headerCellContent}>
              <span>{renderedTitle}</span>
              <font-awesome-icon
                class={$style.sortIcon}
                icon={sortIcon}
                on={{ click: () => this.toggleSort(column.prop) }}
              />
              <FilterDropdown
                class={$style.filterIcon}
                columnProp={column.prop}
                shown={column.prop === filterProp}
                filterText={filterText}

                on={{
                  openFilterTooltip: () => this.openFilterTooltip(column.prop),
                  closeFilterTooltip: () => this.openFilterTooltip(),
                  setFilterText: this.setFilterText,
                }}
              />
            </div>
          </th>
        );
      });
    },
    renderRows(h, columnsOptions) {
      return this.rows.map((row, index) => {
        return <tr key={row.id || index}>{...this.renderColumns(h, row, columnsOptions)}</tr>;
      });
    },
    renderColumns(h, row, columnsOptions) {
      return columnsOptions.map((column) => {
        return (
          <td key={column.prop} class={this.$style.cell}>
            {column.scopedSlots.body ? column.scopedSlots.body({ row }) : row[column.prop]}
          </td>
        );
      });
    },
    getColumnOptions() {
      return this.$slots.default.
        filter(item => item.componentOptions && item.componentOptions.tag === 'oz-table-column').
        map(column =>
          Object.assign({}, column.componentOptions.propsData, {
              scopedSlots: column.data.scopedSlots || {}
            }
          )
        );
    },
    renderInfPager() {
      const directives = [
        {
          name: 'detect-viewport',
          value: {
            callback: this.$listeners.getPage
          }
        }
      ];

      const style = {
        background: `url("${DotsLoaderIcon}") no-repeat center`
      };

      return (
        <div {...{ class: this.$style.infPager, style, directives }} />
      );
    }
  },
  render(h) {
    const { $style, totalPages, currentPage, staticPaging, $listeners } = this;
    const { getPage } = $listeners;
    const columnsOptions = this.getColumnOptions();
    const columnsHead = this.renderHead(h, columnsOptions);
    const rows = this.renderRows(h, columnsOptions);

    return (
      <div>
        <table class={$style.table}>
          <thead>{...columnsHead}</thead>
          <tbody>{...rows}</tbody>
        </table>

        {staticPaging
          ? <OzTablePaginator totalPages={totalPages} currentPage={currentPage} on={{ getPage: getPage }} />
          : this.renderInfPager()
        }

      </div>
    );
  }
};
</script>

<style module>

  .table {
    border-spacing: 0;
    margin: 8px;
    width: 100%;
  }


  .cell {
    text-align: left;
    border-bottom: 1px solid #c8cacc;
    padding: 1rem 1rem;
  }

  .headerCell {
    composes: cell;
    background: #c7cbcb;
  }

  .infPager {
    width: 100%;
    height: 32px;
  }

   .headerCellContent {
    display: flex;
    align-items: center;
  }

  .sortIcon {
    margin-left: 8px;
    margin-right: 24px;
  }

  .sortIcon:hover {
    cursor: pointer;
  }
  
  th[data-key='id'] .filterIcon,
  th[data-key='status'] .filterIcon{
    display: none;
  }
</style>
