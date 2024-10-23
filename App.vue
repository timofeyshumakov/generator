<template>
  <main class="main">
    <div v-if="isLoading">Загрузка...</div>
    <div v-else-if="generating" class="generating">
      <h2>Идет генерация сущностей</h2>
    </div>
    <div v-else class="main-container">
      <h1>Генератор сущностей</h1>
      <v-dialog v-model="dialogSucces" max-width="600">
        <v-card>
          <v-card-title>
            <span class="headline">Успешно</span>
          </v-card-title>
          <v-card-text>
            <p>Заданные сущности успешно созданы</p>
          </v-card-text>
          <v-card-actions>
            <v-btn color="green" text @click="dialogSucces = false">Закрыть</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog v-model="dialog" max-width="600">
        <v-card>
          <v-card-title>
            <span class="headline">Ошибка</span>
          </v-card-title>
          <v-card-text>
            <p>Произошла ошибка при выполнении операции. Пожалуйста, попробуйте еще раз.</p>
          </v-card-text>
          <v-card-actions>
            <v-btn color="red" text @click="dialog = false">Закрыть</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog v-model="dialogSelect" max-width="600">
        <v-card>
          <v-card-title>
            <span class="headline">Ошибка</span>
          </v-card-title>
          <v-card-text>
            <p>Выберите создаваемые сущности</p>
          </v-card-text>
          <v-card-actions>
            <v-btn color="red" text @click="dialogSelect = false">Закрыть</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-form>
        <v-select v-model="selectedUsers" :items="users" label="Выберите пользователей" item-title="FULL_NAME" item-value="ID" multiple chips clearable>
          <template v-slot:prepend-item>
            <v-list-item @click="">
              <v-list-item-content>
                <v-list-item-title>
                  <v-checkbox :input-value="allSelected" label="Выбрать всех пользователей" v-model="selectAllUsers" @change="toggleSelectAll" /> </v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </template>
        </v-select>
        <v-select v-model="selected" :items="items" label="Выберите создаваемые сущности" @update:modelValue="showPannel" multiple chips clearable>
          <template v-slot:prepend-item>
            <v-list-item @click="">
              <v-list-item-content>
                <v-list-item-title>
                  <v-checkbox :input-value="1" label="Выбрать все сущности" v-model="selectAllObjects" @change="toggleSelectObjects" /> </v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </template>
        </v-select>
        <v-text-field v-model="value" type="number" label="Введите число создаваемых сущностей" min="1" max="99" prepend-icon="mdi-arrow-down" append-icon="mdi-arrow-up" @keydown.up.prevent="increment" @keydown.down.prevent="decrement" @input="onInput">
          <template #prepend>
            <v-btn @click="decrement" icon>
              <v-icon icon="minus-circle" color="primary" /> </v-btn>
          </template>
          <template #append>
            <v-btn @click="increment" icon>
              <v-icon icon="plus-circle" color="primary" /> </v-btn>
          </template>
        </v-text-field>
        <div class="form-buttons">
          <v-checkbox label="Связывать сущности" v-model="bindObjects"></v-checkbox>
          <v-btn color="primary" @click="generateData">Создать сущности</v-btn>
        </div>
      </v-form>
      <v-expansion-panels multiple v-model="panel">
        <template v-for="i in this.fields.length">
          <v-expansion-panel class="panel" :disabled="disabled[i - 1]" :value="i">
            <v-expansion-panel-title> {{ this.items[i - 1] }} </v-expansion-panel-title>
            <v-expansion-panel-text>
              <div class="buttons">
                <v-btn color="primary" @click="selectAll(i - 1)" :data-fieldId='i - 1'>Выбрать все</v-btn>
                <v-btn color="primary" @click="clearAll(i - 1)" :data-fieldId='i - 1'>Убрать все</v-btn>
              </div>
              <v-container fluid class="inputs">
                <template v-for="(item, key) in Object.entries(this.fields[i - 1])">
                  <v-switch v-if="item[1].isReadOnly === false && item[1].type !== 'f' && item[1].type.slice(0, 3) !== 'rest'" v-model="selectedFields[i - 1]" color="primary" :label="item[1].title.slice(0, 2) === 'UF' ? item[1].listLabel : item[1].title" :value="item[0]" :disabled="item[1].isRequired === true"></v-switch>
                </template>
              </v-container>
            </v-expansion-panel-text>
          </v-expansion-panel>
        </template>
      </v-expansion-panels>
    </div>
  </main>
</template>

<script>
  import img from './img.json'
  import {
    ref
  }
  from 'vue';
  export default {
    setup() {
        const isLoading = ref(true);
        const generating = ref(false);

        const toggleLoading = () => {
          isLoading.value = false;
        }

        const generatingOff = () => {
          generating.value = false;
        }

        const generatingOn = () => {
          generating.value = true;
        }

        return {
          isLoading,
          generating,
          toggleLoading,
          generatingOff,
          generatingOn,
        };
      },
      data() {
        return {
          value: 1,
          bindObjects: false,
          fields: [],
          oldObjects: [],
          newObjects: {},
          selectedFields: [],
          requiredFields: [],
          selectedUsers: [],
          allFields: [],
          types: [],
          panel: [],
          users: [],
          disabled: [true, true, true],
          items: ['Компании', 'Сделки', 'Контакты'],
          currencies: [],
          statuses: [],
          objects: [],
          selectAllUsers: false,
          selectAllObjects: false,
          selected: null,
          dialog: false,
          dialogSelect: false,
          dialogSucces: false,
        };
      },
      computed: {
        allSelected() {
          return this.selectedUsers.length === this.users.length;
        },
        allSelectedObjects() {
          return this.selected.length === this.items.length;
        }
      },
      methods: {
        toggleSelectAll() {
          if (this.allSelected) {
            this.selectedUsers = [];
          } else {
            this.selectedUsers = [...this.users.map(obj => obj.ID)];
          }
        },
        toggleSelectObjects() {
          if (this.selectAllObjects) {
            this.selected = this.items;
            this.selected.forEach((_, i) => this.panel[i] = i + 1);
            this.disabled = this.disabled.map(() => false);
          } else {
            this.selected = [];
            this.panel = [];
            this.disabled = this.disabled.map(() => true);
          }
        },
        showPannel(value) {
          Array.from(document.getElementsByClassName("panel")).forEach((el, i) => {
            if (value.includes(this.items[i])) {
              this.disabled[i] = false;
              this.panel[i] = i + 1;
            } else {
              this.panel[i] = null;
              this.disabled[i] = true;
            }
          });
        },
        generateData() {

          if(this.selected === null || this.selected.length === 0){
            this.dialogSelect = true;
            return;
          }

          const year = 365.25 * 24 * 60 * 60 * 1000;
          let data = [];
          let d = 0;
          let val;
          let dictionary;
          let methods = [];
          for (let o = 0; o < this.value; o++) {
            this.selectedFields.forEach((obj, arrIndex) => {
              if (this.selected.includes(this.items[arrIndex])) {
                data.push({});
                const gender = this.items[arrIndex] === 'Контакты' ? this.randomWord([
                  ['m', 'w']
                ], 1, '') : null;
                const country = this.randomWord([
                  ['Россия', 'Беларусь']
                ], 1, '');
                Object.entries(this.fields[arrIndex]).map((el, elIndex) => {
                  if (this.selectedFields[arrIndex].includes(el[0])) {
                    const key = el[0];
                    let date;
                    if (el[1].isReadOnly) {
                      return;
                    }
                    switch (this.items[arrIndex]) {
                      case 'Компании':
                        switch (key) {
                          case "TITLE":
                            dictionary = [
                              ['Зенит', 'БрайтСтар', 'Терра', 'Аква', 'Решения', 'Стелларшифт', 'БиоСкайп', 'Солатек'],
                              ['Холдингс', 'Динамика', 'Сфера', 'Технологии', 'Лабс']
                            ];
                            data[d][key] = this.randomWord(dictionary, 2, ' ');
                            break;
                          default:
                            val = this.deafaultField(el[1].type, key, el[1], country);
                            data[d][key] = val !== undefined ? val : null;
                            break;
                        }
                        break;
                      case 'Сделки':
                        switch (key) {
                          case "TITLE":
                            dictionary = ['ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'];
                            data[d][key] = 'Сделка ' + this.randomWord(dictionary, 5, '');
                            break;
                          case "PROBABILITY":
                            data[d][key] = this.randomNumber(2, "inteder", 0);
                            break;
                          case "OPPORTUNITY":
                            data[d][key] = this.randomNumber(5, "inteder", 0);
                            break;
                          case "TAX_VALUE":
                            data[d][key] = (data[d].OPPORTUNITY * 0.13).toFixed(2);
                            break;
                          case "BEGINDATE":
                            data[d][key] = new Date(Date.now()).toISOString().replace(".000Z", "").split('T')[0];
                            break;
                          case "CLOSEDATE":
                            date = this.randomNumber(0, "integer", 0, Math.floor(Date.now() / 1000), Math.floor((Date.now() + year) / 1000));
                            data[d][key] = new Date(date * 1000).toISOString().replace(".000Z", "").split('T')[0];
                            break;
                          default:
                            val = this.deafaultField(el[1].type, key, el[1]);
                            data[d][key] = val !== undefined ? val : null;
                            break;
                        }
                        break;
                      case 'Контакты':
                        switch (key) {
                          case "NAME":
                            if (gender === 'm') {
                              dictionary = [
                                ['Андрей', 'Алексей', 'Иван']
                              ];
                            } else {
                              dictionary = [
                                ['Александра', 'Мария', 'Матильда']
                              ];
                            }
                            data[d][key] = this.randomWord(dictionary, 1, ' ');
                            break;
                          case "SECOND_NAME":
                            if (gender === 'm') {
                              dictionary = [
                                ['Андреев', 'Алексеев', 'Иванов']
                              ];
                            } else {
                              dictionary = [
                                ['Андреева', 'Алексеева', 'Иванова']
                              ];
                            }
                            data[d][key] = this.randomWord(dictionary, 1, ' ');
                            break;
                          case "LAST_NAME":
                            if (gender === 'm') {
                              dictionary = [
                                ['Андреевич', 'Алексеевич', 'Иванов']
                              ];
                            } else {
                              dictionary = [
                                ['Андреевна', 'Алексеевна', 'Ивановна']
                              ];
                            }
                            data[d][key] = this.randomWord(dictionary, 1, ' ');
                            break;
                          case "BIRTHDATE":
                            const max = Math.floor((Date.now() - (18 * year)) / 1000);
                            const maxAge = Math.floor((Date.now() - (65 * year)) / 1000);
                            const min = Math.max(maxAge, 0);
                            date = this.randomNumber(0, "integer", 0, min, max);
                            data[d][key] = new Date(date * 1000).toISOString().replace(".000Z", "").split('T')[0];
                            break;
                          case "POST":
                            dictionary = [
                              ['Сотрудник', 'Старший сотрудник', 'Секретарь', 'Директор']
                            ];
                            data[d][key] = this.randomWord(dictionary, 1, '');
                            break;
                          default:
                            val = this.deafaultField(el[1].type, key, el[1], country);
                            data[d][key] = val !== undefined ? val : null;
                            break;
                        }
                      default:
                        break;
                    }
                  }
                })
                switch (data[d].ADDRESS_CITY) {
                  case 'Москва':
                    data[d].ADDRESS_POSTAL_CODE = `12${this.randomNumber(4)}`;
                    data[d].ADDRESS_PROVINCE = 'Московская область';
                    break;
                  case 'Санкт-Петербург':
                    data[d].ADDRESS_POSTAL_CODE = `18${this.randomNumber(4)}`;
                    data[d].ADDRESS_PROVINCE = 'Ленинградская область';
                    break;
                  case 'Минск':
                    data[d].ADDRESS_POSTAL_CODE = `22${this.randomNumber(4)}`;
                    data[d].ADDRESS_PROVINCE = 'Минская область';
                    break;
                  case 'Могилев':
                    data[d].ADDRESS_POSTAL_CODE = `21${this.randomNumber(4)}`;
                    data[d].ADDRESS_PROVINCE = 'Могилевская область';
                  default:
                    break;
                }
                switch (this.items[arrIndex]) {
                  case 'Компании':
                    methods[d] = 'company';
                    break;
                  case 'Сделки':
                    methods[d] = 'deal';
                    break;
                  case 'Контакты':
                    methods[d] = 'contact';
                    data[d].HONORIFIC = gender === 'm' ? 'HNR_RU_1' : 'HNR_RU_2';
                    break;
                  default:
                    break;
                }
                d++;
              }
            })
          }
          this.generating = true;
          this.handleSubmit(data, methods);
        },
        deafaultField(type, key, obj, country) {
          let dictionary;
          let date;
          switch (key) {

            // устаревшие поля
            case "REG_ADDRESS":
            case "REG_ADDRESS_CITY":
            case "REG_ADDRESS_2":
            case "ADDRESS_2":
            case "ADDRESS_LEGAL":
            case "REG_ADDRESS_COUNTRY":
            case "REG_ADDRESS_COUNTRY_CODE":
            case "REG_ADDRESS_REGION":
              return null;
            //

            case "BANKING_DETAILS":
              return `Расчетный счет: ${this.randomNumber(20)} \n БИК: ${this.randomNumber(9)} \n Корреспондентский счет: ${this.randomNumber(20)}`;
            case "ADDRESS":
              dictionary = [
                ['Грузинский переулок', 'Васильевская улица', 'улица Фадеева']
              ];
              return `${this.randomWord(dictionary, 1, '')}, ${this.randomNumber(2, "", 0, 0, 0)}`;
            case "ADDRESS_CITY":
              switch (country) {
                case 'Россия':
                  dictionary = [
                    ['Москва', 'Санкт-Петербург']
                  ];
                  return this.randomWord(dictionary, 1, '');
                case 'Беларусь':
                  dictionary = [
                    ['Могилев', 'Минск']
                  ];
                  return this.randomWord(dictionary, 1, '');
                default:
                  break;
              }
            case "ADDRESS_COUNTRY":
              return country;
            case "ADDRESS_COUNTRY_CODE":
              switch (country) {
                case 'Россия':
                  return 'RU';
                case 'Беларусь':
                  return 'BY';
                default:
                  break;
              }
            case "ADDRESS_REGION":
              dictionary = [
                ['Пресненский район', 'Красносельский район', 'Нижегородский район']
              ];
              return this.randomWord(dictionary, 1, '');
            default:
              //if(data[d][key]){ return; }
              switch (type) {
                case "string":
                  return this.randomWord('', 10, '');
                case "integer":
                  return this.randomNumber(5, "integer");
                case "double":
                  if (obj.settings) {
                    let precision = obj.settings.PRECISION;
                    const length = obj.settings.SIZE;
                    const min = obj.settings.MIN_VALUE;
                    const max = obj.settings.MAX_VALUE;
                    return this.randomNumber(length, precision, min, max);
                  } else {
                    const length = 5;
                    return this.randomNumber(length, "double");
                  }
                case "enumeration":
                  const variants = obj.items.map(item => item.ID);
                  return variants[this.randomNumber(0, "integer", 0, 0, variants.length - 1)];
                case "user":
                  return this.selectedUsers[this.randomNumber(0, "integer", 0, 0, this.selectedUsers.length - 1)];
                case "datetime":
                  date = this.randomNumber(0, "integer", 0, 0, Math.floor(Date.now() / 1000));
                  return new Date(date * 1000).toISOString().replace(".000Z", "");
                case "date":
                  date = this.randomNumber(0, "integer", 0, 0, Math.floor(Date.now() / 1000));
                  return new Date(date * 1000).toISOString().replace(".000Z", "").split('T')[0];
                case "char":
                  return !!this.randomNumber(0, "integer", 0, 0, 1);
                case "boolean":
                  return !!this.randomNumber(0, "integer", 0, 0, 1);
                case "location":
                  dictionary = ['ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'];
                  return this.randomWord(dictionary, 10, '');
                case "file":
                  return {
                    "fileData": ["1.png", img.img[this.randomNumber(0, "integer", 0, 0, img.img.length - 1)]]
                  };
                case "crm_multifield":
                  switch (key) {
                    case "PHONE":
                      const areaCode = Math.floor(99 + Math.random() * 900);
                      const firstPart = Math.floor(99 + Math.random() * 900);
                      const secondPart = Math.floor(9 + Math.random() * 90);
                      const thirdPart = Math.floor(9 + Math.random() * 90);
                      return [{
                        'VALUE': `+7${areaCode}${firstPart}${secondPart}${thirdPart}`,
                        'VALUE_TYPE': 'HOME'
                      }];
                    case "EMAIL":
                      return [{
                        'VALUE': `${this.randomWord(dictionary, 10, '')}@mail.ru`,
                        'VALUE_TYPE': 'HOME'
                      }];
                    case "WEB":
                      return [{
                        'VALUE': `${this.randomWord(dictionary, 10, '')}.ru`
                      }];
                    case "IM":
                      return [{
                        'VALUE': `https://t.me/${this.randomWord(dictionary, 10, '')}`,
                        'VALUE_TYPE': 'Telegram'
                      }];
                    case "LINK":
                      return [{
                        'VALUE': `${this.randomWord(dictionary, 10, '')}.ru`
                      }];
                    default:
                      return this.randomWord(dictionary, 10, '');
                  }
                case "crm_currency":
                  return this.currencies[this.randomNumber(0, "integer", 0, 0, this.currencies.length - 1)].CURRENCY;
                case "crm_status":
                  const result = this.statuses.filter(searchObj => searchObj.ENTITY_ID === obj.statusType).map(searchObj => searchObj.STATUS_ID);
                  return result[this.randomNumber(0, "", 0, 0, result.length)];
                case "crm_lead":
                  return this.oldObjects[3][this.randomNumber(0, "", 0, 0, this.oldObjects[3].length)];
                case "crm_contact":
                  if (this.bindObjects) {
                    return this.oldObjects[2][this.randomNumber(0, "", 0, 0, this.oldObjects[2].length)];
                  }
                case "crm_company":
                  if (this.bindObjects) {
                    return this.oldObjects[0][this.randomNumber(0, "", 0, 0, this.oldObjects[0].length)];
                  }
                default:
                  break;
              }
              break;
          }
        },
        randomWord(dictionary, length, template) {
          if (!dictionary) {
            dictionary = ['ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'];
          }

          let result = [];
          if (!length) {
            length = dictionary.length;
          }

          for (let a = 0; a < length; a++) {
            let randomIndex;
            if (a < dictionary.length) {
              randomIndex = Math.floor(Math.random() * dictionary[a].length);
              result[a] = dictionary[a][randomIndex];
            } else {
              randomIndex = Math.floor(Math.random() * dictionary[a % dictionary.length].length);
              result[a] = dictionary[a % dictionary.length][randomIndex];
            }
          }
          return result.join(template);
        },
        randomNumber(length, type, precision, min, max) {
          if (min === 0, max === 1) {
            return Math.round(Math.random());
          }!precision && type === "double" ? precision = 2 : precision = 0;

          let actualMin;
          if (length && min) {
            actualMin = min;
          } else if (length && !min) {
            actualMin = Math.pow(10, length - 1);
          } else if (!length && min) {
            actualMin = min;
          } else {
            actualMin = 0;
          }

          if (max) {
            if (length > (max.toString().length) || length < 1) {
              length = max.toString().length;
            }
          }

          const maxLengthNumber = Math.pow(10, length) - 1;
          const actualMax = Math.min(max ? max : maxLengthNumber, maxLengthNumber);
          if (actualMin > actualMax) {
            return actualMax;
          }

          const formatedNumber = Math.floor((Math.random() * (actualMax - actualMin)) + actualMin).toFixed(precision);
          return formatedNumber;
        },
        selectAll(objectIndex) {
          this.selectedFields[objectIndex] = this.allFields[objectIndex]
        },
        clearAll(objectIndex) {
          this.selectedFields[objectIndex] = this.requiredFields[objectIndex];
        },
        onInput() {
          if (this.value.length > 2) {
            this.value = this.value.slice(0, 2);
          }
          if (this.value > 50) {
            this.value = 50;
          }
          if (this.value < 1) {
            this.value = 1;
          }
        },
        increment() {
          if (this.value < 50) {
            this.value++;
          }
        },
        decrement() {
          if (this.value > 1) {
            this.value--;
          }
        },
        async handleSubmit(data, methods) {
          const maxTotal = 50;
          try {
            if (data.length === 1) {
              await new Promise((resolve, reject) => {
                BX24.callMethod(`crm.${methods[0]}.add`, {
                  fields: data[0],
                }, (res) => {
                  if (res.data()) {
                    resolve();
                  }
                });
              });
            } else if (data.length > 1) {
              let cmd = {};
              for (let i = 0; i < data.length; i++) {
                const key = `cmd${i}`;
                const value = {
                  method: `crm.${methods[i]}.add`,
                  params: {
                    fields: data[i]
                  }
                };
                cmd[key] = value;
                if (!this.newObjects[methods[i]]) {
                  this.newObjects[methods[i]] = [];
                }
                if ((i + 1) % maxTotal === 0 || i + 1 === data.length) {
                  const batchLength = (i + 1) % maxTotal === 0 ? maxTotal : (data.length % maxTotal);
                  await new Promise((resolve, reject) => {
                    BX24.callBatch(cmd, (res) => {
                      for (let r = i - batchLength + 1; r < i + 1; r++) {
                        this.newObjects[methods[r]].push(res[`cmd${r}`].data());
                      }
                      resolve();
                    });
                  })
                  cmd = {};
                }
              }
            }
            await this.updateObjects(this.newObjects);
            this.dialogSucces = true;
            this.generating = false;
          } catch (error) {
            this.dialog = true;
            this.generating = false;
          }
        },
        async updateObjects(objects) {
          let cmd = {};
          let totalCount = 0;
          let i = 0;
          let obj = 0;
          for (const key in objects) {
            totalCount += objects[key].length;
          }
          for (const key in objects) {
            objects[key].forEach((el, index) => {
              let fields;
              const contactId = this.bindObjects ? `${objects.contact[this.randomNumber(0, '', 0, 0, objects.contact.length - 1)]}` : this.oldObjects[2][this.randomNumber(0, "", 0, 0, this.oldObjects[2].length)];
              const leadId = this.oldObjects[3][this.randomNumber(0, "", 0, 0, this.oldObjects[3].length)];
              const companyId = this.bindObjects ? `${objects.company[this.randomNumber(0, '', 0, 0, objects.company.length - 1)]}` : this.oldObjects[0][this.randomNumber(0, "", 0, 0, this.oldObjects[0].length)];
              const id = el;
              const objectName = Object.keys(objects)[obj];
              switch (objectName) {
                case 'company':
                  fields = {
                    "LEAD_ID": leadId  //связанный контакт задается только через сущность контакт
                  };
                  break;
                case 'contact':
                  fields = {
                    "COMPANY_ID": companyId,
                    "LEAD_ID": leadId
                  };
                case 'deal':
                  fields = {
                    "COMPANY_ID": companyId,
                    "LEAD_ID": leadId,
                    'CONTACT_ID': contactId
                  };
                  break;
                default:
                  break;
              }
              const key = `cmd${i}`;
              const value = {
                method: `crm.${objectName}.update`,
                params: {
                  id: id,
                  fields: fields,
                }
              };
              cmd[key] = value;
              if ((i + 1) % 50 === 0 || i + 1 === totalCount) {
                i > 50 ? setTimeout(() => this.updateFunction(totalCount, cmd), 2000) : this.updateFunction(totalCount, cmd);
                cmd = {};
              }
              i++;
            });
            obj++;
          }
        },
        async updateFunction(totalCount, cmd) {
          try{
            if (totalCount === 1) {
              await new Promise((resolve, reject) => {
                BX24.callMethod(`crm.${Object.keys(objects)[0]}.update`, {
                  id: cmd.cmd0.id,
                  fields: cmd.cmd0.fields,
                }, (res) => {
                  if (res.data()) {
                    resolve();
                  }
                });
              });
            } else {
              await new Promise((resolve, reject) => {
                BX24.callBatch(cmd, (res) => {
                  resolve();
                });
              })
            }
          } catch (error) {
            this.dialog = true;
            this.generating = false;
          } 
        },
        async callApi(method, filter, select) {
          if (!filter) {
            filter = {};
          }
          let total = 0;
          const maxTotal = 50;
          let data = {};
          const exceptions = ['crm.status.list'];
          await new Promise((resolve, reject) => {
            BX24.callMethod(method, {
              filter: filter,
              select: select,
            }, (res) => {
              if (res.data()) {
                total = res.total();
                data = res.data();
                resolve(data);
              }
            });
          });
          if (total > maxTotal && !exceptions.includes(method)) {
            let cmd = {};
            const iterations = Math.ceil(total / maxTotal);
            for (let i = 0; i < iterations; i++) {
              const key = `cmd${i}`;
              const value = {
                method: method,
                params: {
                  start: i * maxTotal,
                  filter,
                  select: select
                }
              };
              cmd[key] = value;
            }
            await new Promise((resolve, reject) => {
              BX24.callBatch(cmd, (res) => {
                let resultData = [];
                for (let i = 0; i < iterations; i++) {
                  let key = `cmd${i}`;
                  data = res[key].data();
                  resultData.push(data);
                }
                resultData = resultData.flat();
                data = resultData;
                resolve(resultData);
              });
            })
          }
          return data;
        },
      },
      async mounted() {
        await new Promise((resolve, reject) => {
          BX24.callBatch({
            get_users: ['user.get', {}],
            get_deals: ['crm.deal.fields', {}],
            get_companies: ['crm.company.fields', {}, ],
            get_contacts: ['crm.contact.fields', {}],
            get_currencies: ['crm.currency.list', {}],
            get_statuses: ['crm.status.list', {}],
            get_companies_list: {
              method: 'crm.company.list',
              params: {
                select: ['ID']
              }
            },
            get_contacts_list: {
              method: 'crm.contact.list',
              params: {
                select: ['ID']
              }
            },
            get_leads_list: {
              method: 'crm.lead.list',
              params: {
                select: ['ID']
              }
            },
          }, (res) => {
            this.users = res.get_users.data();
            this.fields[0] = res.get_companies.data();
            this.fields[1] = res.get_deals.data();
            this.fields[2] = res.get_contacts.data();
            this.currencies = res.get_currencies.data();
            this.statuses = res.get_statuses.data();
            this.oldObjects[0] = res.get_companies_list.data().map(obj => obj.ID);
            //индекс 1 зарезервирован под сделки
            this.oldObjects[2] = res.get_contacts_list.data().map(obj => obj.ID);
            this.oldObjects[3] = res.get_leads_list.data().map(obj => obj.ID);
            resolve();
          });
        });

        for (let i = 0; i < this.items.length; i++) {
          this.selectedFields.push([]);
          this.requiredFields.push([]);
          this.allFields.push([]);
          this.types.push([]);
        }

        let r = 0;
        this.fields.forEach((obj, oi) => {
          Object.entries(obj).map((key, i) => {
            this.types[oi][i] = key[1].type;
            if (!key[1].isReadOnly) {
              this.allFields[oi][i] = key[0];
            }
            if (key[1].isRequired) {
              this.requiredFields[oi][r] = key[0];
              this.selectedFields[oi][i] = key[0];
              r++;
            }
          });
        })

        this.users.forEach(user => {
          user.FULL_NAME = `${user.SECOND_NAME ? user.SECOND_NAME : ''} ${user.NAME ? user.NAME : ''} ${user.LAST_NAME ? user.LAST_NAME : ''}`;
        });

        this.toggleLoading();

      }
  }
</script>

<style lang="sass">
  
  ::-webkit-scrollbar
    background: white
  
  ::-webkit-scrollbar-thumb
    background: #e6e6e6
    border-radius: 1rem
  
  ::-webkit-scrollbar-thumb:active
    background: #e1e1e1
  
  html, body
    min-height: 100%

  h1
    width: 100%
    text-align: center

  .main-container
    display: flex
    flex-direction: column
    gap: 0.75rem
    padding: 0.5rem
  
  .inputs
    display: grid
    grid-template-columns: repeat(auto-fit, minmax(20rem, 1fr))
    column-gap: 1rem
    padding: 0
  
  .buttons
    margin-bottom: 1rem
    display: grid
    grid-template-columns: repeat(4, 1fr)
    gap: 1rem
    width: 100%
  
  .buttons button
    width: 100%
  
  .period
    min-width: 12rem
    max-width: 12rem
  
  .charts
    grid-template-columns: repeat(2, 1fr)
    gap: 0.75rem
  
  .chart
    border: 1px solid black
    border-radius: 1rem
  
  .header
    margin: 1.5rem 0
    display: flex
    gap: 1rem
    width: 100%
    justify-content: space-between
    align-items: center
  
  .tfoot__td:not(:first-child)
    text-align: center
  
  [type=number]::-webkit-inner-spin-button
    display: none
  
  .v-input__prepend .v-icon:not(.v-btn__content .v-icon)
    display: none
  
  .v-input__append .v-icon:not(.v-btn__content .v-icon)
    display: none
  
  .v-btn--icon .v-btn__content
    transform: scale(2)
  
  .v-data-table-footer
    display: none
  
  .v-input__details
    display: none
  
  .v-btn.v-btn--density-default
    height: 3rem
  
  .v-form
    width: 100%
    display: grid
    grid-template-columns: 1fr 1fr
    gap: 0.75rem
  
  
  .v-form > *:not(.v-checkbox)
    min-width: 25rem
    flex-grow: 1
  
  .v-checkbox .v-input__control
    min-width: unset
  
  .v-btn--size-default.v-btn
    min-width: unset
    max-width: 20rem
  
  .form-buttons
    width: 100%
    display: flex
    gap: 1.25rem

  .generating
    width: 100%
    height: 100%
    display: flex
    flex-direction: column
    align-items: center
    justify-content: center
    gap: 1rem

  .generating-img
    max-width: 30rem

</style>
  
  
  
