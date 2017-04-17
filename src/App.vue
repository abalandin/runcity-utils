<template>
  <div id="app">
    <b-card>
      <b-form-fieldset>
        <b-form-radio v-model="filter.okato"
                      :options="refs.okato">
        </b-form-radio>
      </b-form-fieldset>
      <b-form-fieldset label="Район"
                       horizontal>
        <b-form-select v-model="filter.okato2"
                       :options="refs.okato2">
        </b-form-select>
      </b-form-fieldset>
      <b-form-fieldset label="Тип улицы"
                       horizontal>
        <b-form-select v-model="filter.streetType"
                       :options="refs.streetType">
        </b-form-select>
      </b-form-fieldset>
      <div class="row">
        <div class="col-3"></div>
        <div class="col-4">
          <b-form-fieldset label="Содержит числительные"
                           :label-size=6
                           horizontal>
            <b-form-checkbox v-model="filter.withNumbers">
            </b-form-checkbox>
          </b-form-fieldset>
        </div>
        <div class="col-4">
          <b-form-fieldset label="Содержит персону"
                           :label-size=6
                           horizontal>
            <b-form-checkbox v-model="filter.withPerson">
            </b-form-checkbox>
          </b-form-fieldset>
        </div>
      </div>

      <b-form-fieldset label="Шаблон названия"
                       horizontal>
        <b-form-input v-model="filter.streetNameTemplate">
        </b-form-input>
      </b-form-fieldset>

      <b-btn variant="primary"
             @click="searchStreets">Применить фильтр</b-btn>

      <b-btn @click="clearForm">Очистить форму</b-btn>
    </b-card>

    <div v-if="foundStreets.items.length > 0"
         class="justify-content-center row my-1">
      <b-pagination size="md"
                    :total-rows="foundStreets.items.length"
                    :per-page="foundStreets.perPage"
                    v-model="foundStreets.currentPage" />
    </div>
    <b-table v-if="foundStreets.items.length > 0"
             striped
             hover
             :items="foundStreets.items"
             :fields="foundStreets.fields"
             :current-page="foundStreets.currentPage"
             :per-page="foundStreets.perPage">
      <template slot="type"
                scope="item">
        {{item.value}}
      </template>
      <template slot="name"
                scope="item">
        {{item.value}}
      </template>
    </b-table>

  </div>
</template>

<script>
import okatoRef from './data/okato.json';
import streetRef from './data/moscow-streets.json';
import peopleRef from './data/people.json';

function okatoItemConverter(item) {
  return {
    value: item.value,
    text: `[${item.value}] ${item.text}`,
  };
}

const anyItem = {
  value: '',
  text: '[все]',
};

const streetTypeRef = [anyItem].concat(Object.getOwnPropertyNames(streetRef.reduce((res, item) => {
  res[item.type] = true;
  return res;
}, {})));

const peopleNameRoots = peopleRef.map(item => item.text.replace(/(а|ий|ая|ого|ь)$/i, ''));

const data = {
  filter: {
    okato: '',
    okato2: '',
    streetType: '',
    withNumbers: false,
    withPerson: false,
  },
  refs: {
    okato: [anyItem].concat(okatoRef
      .filter(item => item.value.length < 6)
      .map(okatoItemConverter)),
    okato2: [
      anyItem,
    ],
    streetType: streetTypeRef,
  },
  foundStreets: {
    currentPage: 1,
    perPage: 10,
    items: [],
    fields: {
      type: {
        label: 'тип',
      },
      name: {
        label: 'наименование',
      },
      okato: {
        label: 'ОКАТО',
      },
    },
  },
};

export default {
  name: 'app',
  data() {
    return data;
  },
  components: {
  },
  watch: {
    'filter.okato': function onOkatoChanged(val) {
      const re = new RegExp(`^${val}`);
      this.filter.okato2 = '';
      this.refs.okato2 = [
        anyItem,
      ].concat(okatoRef
        .filter(item => item.value !== val && re.test(item.value))
        .map(okatoItemConverter));
    },
  },
  methods: {
    searchStreets() {
      const app = this;
      const okato = app.filter.okato2 || app.filter.okato;
      const re = new RegExp(`^${okato}`);

      function filterFn(item) {
        if (item.active !== '1') {
          return false;
        }
        if (okato && !re.test(item.okato)) {
          return false;
        }
        if (app.filter.streetType && app.filter.streetType !== item.type) {
          return false;
        }
        if (app.filter.withNumbers && !/\d/.test(item.name)) {
          return false;
        }
        if (app.filter.withPerson) {
          if (!peopleNameRoots.some(person => item.name.indexOf(person) > -1)) {
            return false;
          }
        }
        if (app.filter.streetNameTemplate) {
          const reName = new RegExp(app.filter.streetNameTemplate.replace('*', '.*'), 'i');
          if (!reName.test(item.name)) {
            return false;
          }
        }
        return true;
      }

      app.foundStreets.items = streetRef.filter(filterFn);
    },
    clearForm() {
      const app = this;
      app.filter.okato2 = '';
      app.filter.okato = '';
      app.filter.streetType = '';
      app.filter.streetNameTemplate = '';
    },
  },
};
</script>

<style>

</style>
