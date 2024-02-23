<template>
    <div class="pulldown">
        <div class="pulldown__circle" v-if="status === 'up' || status === 'down'">
            <div class="pulldown_icon_box" :style="{ transform: `scale(${scale})`}">
                <div class="pulldown_icon_point_box" :class="{loading: status === 'loading'}" :style="{ transform: `rotate(${-angle}deg)` }">
                    <div class="pulldown_icon_point" ref="loadingPoint"></div>
                </div>
                <img class="rotate-img" width="40" height="40" src="./icon_g7_refresh_new.png">
            </div>
        </div>
        <div class="pulldown__loading" v-show="status === 'loading'">
            <spinner type="ios"></spinner>
        </div>
    </div>
</template>
<script lang="babel">
    import { XCircle, Spinner } from 'vux';

    export default {
        props: ['status', 'gap'],
        components: {
            XCircle,
            Spinner,
        },
        data() {
            return {};
        },
        computed: {
            percent() {
                const per = -this.gap / 0.6;

                return Math.max(0, Math.min(per, 100));
            },
            angle() {
                return this.percent / 200 * 360;
            },
            scale() {
                const scale = 0.5 + this.percent / 200;
                return scale > 1 ? 1 : scale;
            },
        },
        mounted() {
            this.initPoint();
        },
        methods: {
            initPoint() {},
        },
    };
</script>
<style lang="less" type="text/less">
    .pulldown {
        position: absolute;
        width: 100%;
        height: 60px;
        line-height: 60px;
        top: -60px;
        text-align: center;

        &__circle,
        &__loading {
            display: inline-block;
            margin-top: 15px;
            width: 40px;
            height: 40px;
            line-height: 40px;
        }

        img {
            display: block;
        }

        path {
            transition: none !important;
        }
        .pulldown_icon_box {
            transform: scale(0.8);
            position: absolute;
        }
        .pulldown_icon_point_box {
            position: absolute;
            width: 100%;
            height: 100%;
            &.loading {
                animation: rotateRound 1000ms infinite linear;
            }
        }
        .pulldown_icon_point {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background-color: #C1B6FB;
            position: absolute;
            left: 50%;
            top: 50%;
            margin-top: -4px;
            margin-left: -4px;
            transform: translateY(-19px);
        }
    }

    @keyframes rotateRound {
        0% {
            transform: rotate(-180deg);
        }
        100% {
            transform: rotate(-540deg);
        }
    }
</style>
