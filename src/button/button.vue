<template>
    <button class="gulu-button" :class="{[`icon-${iconPosition}`]: true}"
            @click="$emit('click')">
        <g-icon v-if="icon && !loading" :name="icon"></g-icon>
        <g-icon v-if="loading" class="loading" name="loading"></g-icon>
        <div class="content">
            <slot/>
        </div>
    </button>
</template>
<script>
    import GIcon from '../icon.vue'

    export default {
        name: 'GuluButton',
        components: {
            'g-icon': GIcon
        },
        // props: ['icon', 'iconPosition']
        props: {
            icon: {},
            loading: {
                type: Boolean,
                default: false
            },
            iconPosition: {
                type: String,
                default: 'left',
                //属性的检查器
                validator(value) {
                    return value === 'left' || value === "right"
                }
            }
        }
    }
</script>
<style lang="scss" scoped>
    @import '../../styles/var';

    .gulu-button {
        font-size: $font-size;height: $button-height;padding: 0 1em;background-color: $button-bg;
        border-radius: $button-radius;border: 1px solid $button-border-color;color: #000;
        vertical-align: middle;display: inline-flex;justify-content: center;align-items: center;

        &:hover {border-color: $button-hover-color;cursor: pointer;}

        &:active {background-color: $button-active-bg;}

        &:focus {outline: none;}

        > .content {order: 2;}

        > .icon {order: 1;margin-left: 0;margin-right: .3em;}

        &.icon-right {
            > .content {order: 1;}

            > .icon {order: 2;margin-left: .3em;margin-right: 0;}
        }

        .loading {@include spin;}

        & + & {
            margin-left: 4px;
        }

    }
</style>