<template>
    <div class="monthly-picker">
        <div class="month-picker-wrapper"
            :class="{ 'active visible':showMenu }"
        >
            <div class="month-year-label picker" type="text" autocomplete="off" tabindex="0" @click="openMenu">
                <div class="month-year-display"
                    :disabled="disabled"
                    :class="[inputClass, {'placeholder': !value}]"
                    @click="openMenu"
                >
                    <div class="display-text" :class="'display-text-'+alignment" :style="[{'text-align': alignment}]">{{ displayText }}</div>
                    <span v-if="clearOption && value" class="vmp-input-append" @click.stop.prevent="clearSelect">
                        <i class="vmp-clear-icon" />
                    </span>
                </div>
            </div>
            <div class="text" />
            <div class="date-popover" :class="menuClass" :style="menuStyle" tabindex="-1">
                <div class="picker" style="text-align: center">
                    <div class="flexbox">
                        <span class="prev" :class="{deactive: !canBack}" @click="prevYear" />
                        <div>{{ year }}{{ yearLabels }}</div>
                        <span class="next" :class="{deactive: !canNext}" @click="nextYear" />
                    </div>
                    <div class="flexbox monthItem">
                        <template v-for="(mnth, idx) in monthLabels">
                            <div v-if="isActive(idx)"
                                :key="idx"
                                class="item active"
                                :class="{'selected': isCurrentSelected(year, idx)}"
                                :style="[{'background-color': getBackgroundColor(year, idx)}]"
                                @click="selectMonth(idx)"
                            >{{ mnth }}
                            </div>
                            <div v-else
                                :key="idx"
                                :class="{'selected': isCurrentSelected(year, idx)}"
                                class="item deactive"
                            >
                                {{ mnth }}
                            </div>
                        </template>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import moment from 'moment';

export default {
    name: 'MonthlyPicker',
    props: {
        value: {
            type: Object,
            default: null
        },
        disabled: {
            type: Boolean,
            default: false
        },
        inputClass: {
            type: String,
            default: 'input'
        },
        placeHolder: {
            type: String,
            default: ''
        },
        alignment: {
            type: String,
            default: 'left',
            validator(value) {
                return ['left',
                    'right',
                    'center'].indexOf(value) !== -1;
            }
        },
        selectedBackgroundColor: {
            type: String,
            default: '#007bff'
        },
        monthLabels: {
            type: Array,
            default() {
                return [
                    '1',
                    '2',
                    '3',
                    '4',
                    '5',
                    '6',
                    '7',
                    '8',
                    '9',
                    '10',
                    '11',
                    '12'
                ];
            }
        },
        yearLabels: {
            type: String,
            default: ''
        },
        min: {
            type: Number,
            default: null
        },
        max: {
            type: Number,
            default: null
        },
        dateFormat: {
            type: String,
            default: 'YYYY/MM'
        },
        clearOption: {
            type: Boolean,
            default: true
        }
    },
    data() {
        return {
            showMenu: false,
            year: moment().format('YYYY'),
            month: moment().format('MM')
        };
    },
    computed: {
        menuClass() {
            return {
                visible: this.showMenu,
                hidden: !this.showMenu
            };
        },
        menuStyle() {
            return {
                display: this.showMenu ? 'block' : 'none',
                left: '50%',
                transform: 'translate(-50%,0)'
            };
        },
        displayText() {
            if (this.value) {
                let valueMoment = null;
                if (typeof this.value === 'string') {
                    valueMoment = moment(this.value);
                } else {
                    valueMoment = this.value;
                }
                if (valueMoment && valueMoment.isValid()) {
                    return valueMoment.format(this.dateFormat);
                }
            }
            return this.placeHolder;
        },
        canBack() {
            if (!this.min) return true;
            const currentVal = this.internalMomentValue.clone().startOf('year');
            return this.min.isBefore(currentVal);
        },
        canNext() {
            if (!this.max) return true;
            const currentVal = this.internalMomentValue.clone().endOf('year');
            return currentVal.isBefore(this.max);
        },
        internalMomentValue() {
            const yrMonth = this.year + '/' + this.month;
            return moment(yrMonth, 'YYYY/MM');
        }
    },
    watch: {
        value(value) {
            this.setValue(value);
        }
    },
    mounted() {
        this.init();
    },
    methods: {
        init() {
            document.addEventListener('click', (e) => {
                if (this.$el && !this.$el.contains(e.target)) {
                    this.closeMenu();
                }
            }, false);
            this.setValue(this.value);
        },
        openMenu() {
            if (!this.disabled) {
                this.showMenu = true;
            }
        },
        closeMenu() {
            this.showMenu = false;
        },
        prevYear() {
            if (!this.canBack) return;
            const newYear = parseInt(this.year, 10) - 1;
            this.year = newYear.toString();
        },
        nextYear() {
            if (!this.canNext) return;
            const newYear = parseInt(this.year, 10) + 1;
            this.year = newYear.toString();
        },
        selectMonth(idx) {
            this.month = (parseInt(idx, 10) + 1).toString();
            this.selectPicker();
            this.closeMenu();
        },
        selectPicker() {
            this.$emit('input', this.internalMomentValue.clone());
            this.$emit('selected', this.internalMomentValue.clone());
        },
        setValue(value) {
            if (typeof value === 'string') {
                moment(value);
            }
            if (value && value.isValid()) {
                this.month = value.format('MM');
                this.year = value.format('YYYY');
            }
        },
        isActive(idx) {
            const realMonth = idx + 1;
            const yrMonth = this.year + '/' + (realMonth < 10 ? '0' + realMonth : realMonth);
            if (this.min && moment(yrMonth, 'YYYY/MM').isBefore(this.min)) {
                return false;
            }
            if (this.max && moment(yrMonth, 'YYYY/MM').isAfter(this.max)) {
                return false;
            }
            return true;
        },
        isCurrentSelected(year, monthIdx) {
            if (!this.value) {
                return false;
            }
            let checkValue = this.value;
            if (typeof this.value === 'string') {
                checkValue = moment(this.value);
            }
            if (checkValue && checkValue.isValid()) {
                const currentMonth = checkValue.format('MM');
                const currentYear = checkValue.format('YYYY');
                return Number(currentMonth) === Number(monthIdx + 1) && Number(currentYear) === Number(year);
            }
            return false;
        },
        getBackgroundColor(year, monthIdx) {
            if (this.isCurrentSelected(year, monthIdx)) {
                return this.selectedBackgroundColor;
            }
            return '';
        },
        clearSelect() {
            this.$emit('input', null);
            this.$emit('selected', null);
        }
    }
};
</script>

<style lang="scss">
$lightgray: #d4d4d4;

.monthly-picker {
  .picker {
    .next,
    .prev {
      &:hover {
        cursor: pointer;
      }
    }

    .monthItem {
      .item {
        border-top: 1px solid $lightgray;
        &.active {
          &:hover {
            cursor: pointer;
            background-color: $lightgray;
          }
        }
      }
    }

    .flexbox {
      padding: 0px;
      display: flex;
      flex-wrap: wrap;
      div {
        flex-grow: 1;
        padding: 15px 0;
      }
      .item {
        flex: 1;
        flex-basis: 25%;
      }
    }
  }

  .placeholder {
    color: #bdbdbd !important;
  }

  .date-popover {
    overflow-x: hidden;
    overflow-y: hidden;
    outline: none;
    max-width: 350px;
    width: 200px;
    border-radius: 0 0 .28571429rem .28571429rem;
    box-shadow: 0 2px 3px 0 rgba(34,36,38,.15);
    background: #fff;
    transition: opacity .1s ease;
    position: absolute;
    margin: auto;
    z-index: 10;
    border: 1px solid $lightgray;
    font-size: 1rem;
    font-weight: 200;
  }

  .month-picker-wrapper {
    position: relative;
    display: block;
    min-width: 100px;
  }

  .month-year-label {
    outline: none;
    .vmp-input-append {
      display: none;
    }
    &:hover .vmp-input-append {
      display: block;
    }
  }
  .text {
    position: relative;
    z-index: 2;
  }
  .month-year-display {
    &:hover {
      cursor: pointer;
    }
  }

  .next,
  .prev {
    width: 16%;
    float: left;
    text-indent: -10000px;
    position: relative;

    &:after {
      content: "";
      position: absolute;
      left: 50%;
      top: 50%;
      -webkit-transform: translateX(-50%) translateY(-50%);
      transform: translateX(-50%) translateY(-50%);
      border: 6px solid transparent;
    }
  }
  .next {
    &:after {
      border-left: 10px solid #000;
      margin-left: 5px;
    }
    &.deactive {
      &:hover {
        cursor: default;
      }
      &:after {
        border-left: 10px solid #999999;
      }
    }
  }

  .prev {
    &:after {
      border-right: 10px solid #000;
      margin-left: -5px;
    }
    &.deactive {
      &:hover {
        cursor: default;
      }
      &:after {
        border-right: 10px solid #999999;
      }
    }
  }

  .input {
    -moz-appearance: none;
    -webkit-appearance: none;
    align-items: center;
    display: inline-flex;
    font-size: 1rem;
    height: 2.25em;
    justify-content: flex-start;
    padding: 2px calc(.625em - 1px);
    position: relative;
    vertical-align: top;
    width: 100%;
    color: #333;
    height: 37px;
    font-size: 1.4rem;
    border-color: transparent;
    border-bottom: 1px solid #d2d2d2;
    cursor: pointer;
    -webkit-box-shadow: inset 0 0px 0px transparent;
    box-shadow: inset 0 0px 0px transparent;
    background-color: transparent;
  }
  .deactive {
    color: #999999;
  }

  .selected {
    color: #fff;
    text-shadow: 0 -1px 0 rgba(0, 0, 0, 0.25);
    font-weight: bold;
  }
  .display-text {
    width: 100%;
    height: 14px;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .display-text-right {
    margin-right: 20px;
  }

  .vmp-input-append {
    position: absolute;
    right: 0;
    width: 30px;
    padding: 6px;
  }
  .vmp-clear-icon {
    display: inline-block;
    width: 100%;
    height: 100%;
    font-style: normal;
    color: #555;
    text-align: center;
    cursor: pointer;
    &:before {
      display: inline-block;
      content: '\2716';
      vertical-align: middle;
    }
  }
}
</style>
