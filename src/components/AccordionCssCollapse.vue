<template>
    <div :id="'collapse-' + parentId" class="accordion-collapse collapse" :aria-labelledby="'heading-' + parentId"
        :data-bs-parent="'#accordion-' + accordionId">
        <div class="accordion-body">
            <div v-for="(selector, selectorIndex) in color.items" :key="selectorIndex">
                <div>
                    <div class="names-block rounded-2 mb-3 small bg-light" :class="{ 'text-danger': selector.remove }">
                        <div class="d-flex align-items-center highlight-toolbar p-1 pe-3 ps-3 border-bottom">
                            <color-picker-select :color-options="listColors()" label="Выберите цвет"
                                empty-option="Без цвета" @input="onSelectedColor(selector, $event)"
                                :selected="findColor(color.color)">
                            </color-picker-select>

                            <strong class="classes ms-3">
                                <div v-for="(name, namesIndex) in selector.names" :key="namesIndex"
                                    :class="{ 'text-danger': name.findClass === false, 'text-success': name.findClass === true }">
                                    {{ name.text }}
                                </div>
                            </strong>
                            <div class="d-flex ms-auto">
                                <!-- Spinner -->
                                <div v-if="selector.findFiles === null" class="spinner-border" role="status">
                                    <span class="visually-hidden">Loading...</span>
                                </div>
                                <button v-else type="button" class="icon text-nowrap" @click.stop="searchFiles(selector)">
                                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor"
                                        class="bi bi-search" viewBox="0 0 16 16">
                                        <path
                                            d="M11.742 10.344a6.5 6.5 0 1 0-1.397 1.398h-.001c.03.04.062.078.098.115l3.85 3.85a1 1 0 0 0 1.415-1.414l-3.85-3.85a1.007 1.007 0 0 0-.115-.1zM12 6.5a5.5 5.5 0 1 1-11 0 5.5 5.5 0 0 1 11 0z" />
                                    </svg>
                                </button>

                                <button type="button" class="icon text-nowrap" @click.stop="remove(selector)">
                                    <svg v-if="selector.remove" xmlns="http://www.w3.org/2000/svg" width="16" height="16"
                                        fill="currentColor" class="bi bi-arrow-left-right" viewBox="0 0 16 16">
                                        <path fill-rule="evenodd"
                                            d="M1 11.5a.5.5 0 0 0 .5.5h11.793l-3.147 3.146a.5.5 0 0 0 .708.708l4-4a.5.5 0 0 0 0-.708l-4-4a.5.5 0 0 0-.708.708L13.293 11H1.5a.5.5 0 0 0-.5.5zm14-7a.5.5 0 0 1-.5.5H2.707l3.147 3.146a.5.5 0 1 1-.708.708l-4-4a.5.5 0 0 1 0-.708l4-4a.5.5 0 1 1 .708.708L2.707 4H14.5a.5.5 0 0 1 .5.5z" />
                                    </svg>
                                    <svg v-else xmlns="http://www.w3.org/2000/svg" width="16" height="16"
                                        fill="currentColor" class="bi bi-trash" viewBox="0 0 16 16">
                                        <path
                                            d="M5.5 5.5A.5.5 0 0 1 6 6v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5Zm2.5 0a.5.5 0 0 1 .5.5v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5Zm3 .5a.5.5 0 0 0-1 0v6a.5.5 0 0 0 1 0V6Z" />
                                        <path
                                            d="M14.5 3a1 1 0 0 1-1 1H13v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V4h-.5a1 1 0 0 1-1-1V2a1 1 0 0 1 1-1H6a1 1 0 0 1 1-1h2a1 1 0 0 1 1 1h3.5a1 1 0 0 1 1 1v1ZM4.118 4 4 4.059V13a1 1 0 0 0 1 1h6a1 1 0 0 0 1-1V4.059L11.882 4H4.118ZM2.5 3h11V2h-11v1Z" />
                                    </svg>
                                </button>
                            </div>
                        </div>
                        <div class="styles-block p-3 pt-2 pb-2 rounded-2">

                            <div class="d-flex flex-column">
                                <div class="d-flex" v-for="(style, styleIndex) in selector.styles" :key="styleIndex">
                                    <div class="me-3 opacity-60">{{ style.line }}:</div>
                                    <div class="flex-grow-1">{{ style.name }}: <span
                                            :class="{ 'old-value': style.newValue }">{{ style.value }}</span><span
                                            v-if="style.newValue" class="new-value">{{ style.newValue }}</span></div>
                                </div>
                            </div>

                        </div>
                        <transition name="fade" mode="out-in">
                            <div class="position-relative" v-if="selector.findFiles">
                                <button type="button" class="btn-close position-absolute"
                                    style="top: 5px; right: 5px; z-index: 10;" aria-label="Close"
                                    @click="selector.findFiles = undefined"></button>
                                <div class="alert alert-danger border-bottom-0 border-start-0 border-end-0" role="alert"
                                    v-if="selector.findFiles && selector.findFiles.length === 0">
                                    Не найден
                                </div>
                                <div v-else>
                                    <div v-for="(item, fileIndex) in selector.findFiles" :key="fileIndex"
                                        class="alert border-0 pt-3 pb-3 border-bottom-0 border-start-0 border-end-0"
                                        :class="{ 'alert-success': item.type === 'exact', 'alert-warning': item.type === 'inexact' }"
                                        role="alert">
                                        <p class="small"><b>{{ item.file }}</b></p>
                                        <div class="d-flex flex-column">
                                            <div v-for="(line, lineIndex) in item.lines" :key="lineIndex" class="d-flex">
                                                <div class="me-3">{{ line.num }}:</div>
                                                <div class="flex-grow-1">{{ line.text }}</div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </transition>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>


<script>
import ColorPickerSelect from './ColorPickerSelect.vue';
import { mapGetters } from "vuex";

export default {
    components: {
        ColorPickerSelect,
    },
    props: {
        parentId: String,
        accordionId: String,
        styleName: String,
        color: Object,
    },
    computed: mapGetters(["theme", "matchColor"]),
    data() {
        return {
            // colors: ,
            selected: null,
            except: [
                'background-clip',
                'border-radius',
                'border-style',
                'background-image',
                'background-repeat',
                'background-position',
                'background-size',
                'background-origin',
                'background-attachment',
                'border-width'
            ],
            exceptItem: ['transparent', 'none']
        }
    },
    methods: {
        onSelectedColor(selector, value) {
            const strict = this.styleName === 'color';

            selector.styles.map(item => {

                if (strict ? item.name === this.styleName : item.name.includes(this.styleName) && !this.except.includes(item.name)) {
                    if (this.exceptItem.includes(item.value)) {
                        return item;
                    }


                    // Попытка на тот же цвет повесить разные переменные закончаться не удачно
                    if (!item.color) {
                        debugger;
                    }
                    if (!this.matchColor.findMatchVarAndColor(this.styleName, item.color.hex, value.name)) {
                        console.log(`Для данного цвета ${item.color.hex} уже зарегистрирована переменная: ${value.name}`);
                    } else {
                        if (value) {
                            item.color.newValue = value.name;
                            item.newValue = item.value.replace(item.color.value, `var(${value.name})`);
                            this.matchColor.add(this.styleName, item);
                        } else {
                            item.color.newValue = undefined;
                            item.newValue = undefined;
                            this.matchColor.remove(this.styleName, item);
                        }
                    }
                }

                return item;
            });
        },
        listColors() {
            return this.theme.getColors(this.styleName);
        },
        findColor(color) {
            const find = this.theme.findColor(this.styleName, color);

            if (find) {
                console.log('Найден цвет с помощью толеранса');
                return find;
            }

            const match = this.matchColor.getValue(this.styleName, color);

            if (match) {
                console.log('Найден цвет из списка уже добавленных цветов');
                return this.theme.findColorByName(this.styleName, match);
            }

            return undefined;
        },
        async searchFiles(selector) {
            selector.findFiles = null;

            let findFiles = [];

            let promises = selector.names.map(item => {
                return this.$store.dispatch('findClasses', item.classes)
                    .then(response => {
                        item.findClass = response.length > 0;
                        return response;
                    });
            });

            Promise.all(promises)
                .then(results => {
                    for (let result of results) {
                        findFiles = findFiles.concat(result);
                    }
                    selector.findFiles = findFiles;
                });
        },
        remove(selector) {
            selector.remove = !selector.remove;
        }
    }
};
</script>

<style scoped>
.names-block {
    border: 1px solid var(--bs-border-color);
}

.names-block>p {
    line-height: 2;
}

.fa-search {
    cursor: pointer;
}

.accordion-body {
    padding-bottom: 0;
}

.styles-block {
    border-top-left-radius: 0 !important;
    border-top-right-radius: 0 !important;
    background-color: var(--bs-body-bg);
}

.selector-style {
    font-size: 0.8rem;
    padding: 0;
    margin: 0;
}

.fade-enter-active,
.fade-leave-active {
    transition: opacity .5s;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}

.fas.fa-search,
.text-lightblue {
    color: lightblue;
}

.fas.fa-search:hover {
    color: rgb(96, 175, 201);
}

.spinner-border {
    color: var(--bs-body-color);
    --bs-spinner-border-width: 1px;
    --bs-spinner-width: 1.2rem;
    --bs-spinner-height: 1.2rem;
    margin: 5px;
}

.text-lightblue {
    color: lightblue;
}

.alert {
    --bs-alert-margin-bottom: 0;
    --bs-alert-border-radius: 0;
}

button.icon {
    display: block;
    padding: 0.5em;
    line-height: 1;
    color: var(--bs-body-color);
    background-color: var(--bd-pre-bg);
    border: 0;
    border-radius: 0.25rem;
}

button.icon:hover {
    color: var(--bs-link-hover-color);
}

.alert-warning p.small {
    margin-bottom: .4rem;
}

.accordion-button::after {
    --bs-accordion-btn-icon-width: 1 rem;
}

.old-value {
    text-decoration: line-through;
}

.new-value {
    margin-left: 5px;
    padding: 0 5px;
    background-color: forestgreen;
    color: white;
}
</style>