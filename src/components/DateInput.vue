<template>
<div :class="fieldClass">
    <input type="number"
        v-model="inputFields[0].name"
        :class="`${fieldClass}__${inputFields[0].name} ${inputClass}`"
        :placeholder="inputFields[0].placeholder"
        :maxlength="inputFields[0].maxlength"
        @keyup="jump($event)"
    />
    <span class="date_input__separator">{{ separator }}</span>
    <input type="number"
        v-model="inputFields[1].name"
        :class="`${fieldClass}__${inputFields[1].name} ${inputClass}`"
        :placeholder="inputFields[1].placeholder"
        :maxlength="inputFields[1].maxlength"
        @keydown.delete="jumpBack($event)"
        @keyup="jump($event)"
    />
    <span class="date_input__separator">{{ separator }}</span>
    <input type="number"
        v-model="inputFields[2].name"
        :class="`${fieldClass}__${inputFields[2].name} ${inputClass}`"
        :placeholder="inputFields[2].placeholder"
        :maxlength="inputFields[2].maxlength"
        @keydown.delete="jumpBack($event)"
    />
</div>
</template>

<script>
import moment from 'moment';

export default {
    props: {
        date: {
            type: String,
            default: ''
        },
        format: {
            type: String,
            default: '',
            validator: function (value) {
                return ['YYYY-MM-DD', 'DD-MM-YYYY', 'MM-DD-YYYY'].indexOf !== -1;
            }
        },
        fieldClass: {
            type: String,
            default: 'date_input'
        },
        inputClass: {
            type: String,
            default: null
        },
        min: {
            type: String,
            default: null
        },
        max: {
            type: String,
            default: null
        },
        separator: {
            type: String,
            default: '-'
        },
    },

    computed: {
        dateObject() {
            const date = this.date
                ? moment(this.date, this.format, true)
                : moment();

            if (date.isInvalid()) {
                console.error('Invalid date given, we recommend using the DD-MM-YYYY or YYYY-MM-DD format.');
            }

            return date;
        },
        day: {
            get() {
                return this.dateObject.getDate();
            },
            set(newValue) {
                this.$emit('input', this.getNewDate(this.year, this.month, newValue));
            }
        },
        month: {
            get() {
                return this.dateObject.getMonth();
            },
            set(newValue) {
                this.$emit('input', this.getNewDate(this.year, newValue, this.day));
            }
        },
        year: {
            get() {
                return this.dateObject.getYear();
            },
            set(newValue) {
                this.$emit('input', this.getNewDate(newValue, this.month, this.day));
            }
        },
        inputFields: {
            const year = { name: 'year', placeholder: 'YYYY', maxlength: 4 };
            const month = { name: 'month', placeholder: 'MM', maxlength: 2 };
            const day = { name: 'day', placeholder: 'DD', maxlength: 2 };

            switch (this.format) {
                case 'YYYY-MM-DD':
                    return [year, month, day];
                case 'MM-DD-YYYY':
                    return [month, day, year];
                default:
                    return [day, month, year];
            }
        }
    },

    data() => ({
        goBack: false,
    }),

    methods() {
        getNewDate(year, month, day) {
            return moment().year(year).month(month).date(day).format(this.format)
        },

        jumpBack(event) {
            const oldLength = event.target.value.length;

            this.goBack = oldLength === 0;
        },

        jump(event) {
            if (event.keyCode || event.charCode) === 8) {
                if (this.goBack) {
                    this.focusPreviousInput(event.target);
                }

                return;
            }

            const newLength = event.target.value.length;
            const maxLength = parseInt(event.target.getAttribute('maxlength'));

            if (newLength === maxLength) {
                this.goToNextInput(event.target);
            }
        },

        focusPreviousInput(element) {
            let previousElement = element;

            while(previousElement) {
                previousElement = previousElement.previousElementSibling;

                if (previousElement && previousElement.tagName === 'input') {
                    this.$nextTick(() => {
                        previousElement.focus();
                    });

                    previousElement = null;
                }
            }
        },

        focusNextInput(element) {
            let nextElement = element;

            while(nextElement) {
                nextElement = nextElement.nextElementSibling;

                if (nextElement && nextElement.tagName === 'input') {
                    this.$nextTick(() => {
                        nextElement.focus();
                    });

                    nextElement = null;
                }
            }
        }
    }
};
</script>
