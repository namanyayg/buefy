<template>
    <td
        v-if="visible"
        :class="rootClasses"
        :data-label="label">
        <span><slot/></span>
    </td>
</template>

<script>
    export default {
        name: 'BTableColumn',
        props: {
            label: String,
            customKey: [String, Number],
            field: String,
            meta: [String, Number, Boolean, Function, Object, Array, Symbol],
            width: [Number, String],
            numeric: Boolean,
            centered: Boolean,
            sortable: Boolean,
            visible: {
                type: Boolean,
                default: true
            },
            customSort: Function,
            internal: Boolean // Used internally by Table
        },
        data() {
            return {
                newKey: this.customKey || this.label
            }
        },
        computed: {
            rootClasses() {
                return {
                    'has-text-right': this.numeric && !this.centered,
                    'has-text-centered': this.centered
                }
            }
        },
        methods: {
            /**
             * Find the correct parent
             */
            getCorrectParent () {
                let parent
                if (!this.$parent.$data._isTable) {
                    if (this.$parent.$parent.customRow) {
                        parent = this.$parent.$parent
                    } else {
                        this.$destroy()
                        throw new Error(`You should wrap bTableColumn on a bTable,
                        or set the custom-row prop as true`)
                    }
                } else {
                    parent = this.$parent
                }
                return parent
            },

            addRefToTable() {
                const $parent = this.getCorrectParent()
                if (this.internal) return

                // Since we're using scoped prop the columns gonna be multiplied,
                // this finds when to stop based on the newKey property.
                const repeated = parent.newColumns.some(
                    (column) => column.newKey === this.newKey)
                !repeated && parent.newColumns.push(this)
            }
        },
        beforeMount() {
            this.addRefToTable()
        },
        beforeUpdate() {
            this.addRefToTable()
        },
        beforeDestroy() {
            let $parent = this.getCorrectParent()

            const index = $parent.newColumns.map(
                (column) => column.newKey).indexOf(this.newKey)
            if (index >= 0) {
                $parent.newColumns.splice(index, 1)
            }
        }
    }
</script>
