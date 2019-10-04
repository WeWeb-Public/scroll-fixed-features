<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div class="scroll-fixed-features">
        <!-- wwManager:start -->
        <wwSectionEditMenu v-bind:sectionCtrl="sectionCtrl"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <!-- Weweb Wallpaper -->
        <wwObject class="background" v-bind:ww-object="section.data.color" ww-category="background"></wwObject>

        <div class="content">
            <div ref="leftContent" class="left-content">
                <div @click="nextFeature">next</div>
                <div @click="previousFeature">previous</div>
                <div class="fixed-left">
                    <div v-for="(feature, index) in features" :key="index" class="feature" :class="{'active': feature.active}">
                        <div class="title">{{feature.title}}</div>
                        <div class="details" :class="{'show': feature.active}">
                            <div v-for="(detail, index) in feature.details" :key="index">{{detail}}</div>
                        </div>
                    </div>
                </div>
            </div>
            <div ref="rightContent" class="right-content">
                <div>#READY MADE COMPONENTS</div>
                <div ref="content_0" :style="{height:'400px', width: '100%', 'background-color': 'blue'}"></div>

                <div ref="content_1" :style="{height:'400px', width: '100%', 'background-color': 'blue', 'margin-top':'300px'}"></div>
                <div ref="content_2" :style="{height:'400px', width: '100%', 'background-color': 'blue', 'margin-top':'300px'}"></div>
            </div>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>
import { setTimeout } from 'timers';
export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            activeFeatureIndex: 0,
            features: [
                {
                    active: false,
                    title: 'BUILD',
                    details: ['Enjoy a library of responsive components ready to be used for your front-end, all open-source'],
                    contents: []
                },
                {
                    active: false,
                    title: 'BUILD',
                    details: ['Enjoy a library of responsive components ready to be used for your front-end, all open-source'],

                },
                {
                    active: false,
                    title: 'BUILD',
                    details: ['Enjoy a library of responsive components ready to be used for your front-end, all open-source'],
                }
            ],
            lastScrollTop: 0,
            onTheMove: false
        }
    },
    computed: {
        section() {
            return this.sectionCtrl.get();
        }

    },
    created() {

        //Initialize section data
        let needUpdate = false

        this.section.data = this.section.data || {};

        if (!this.section.data.color) {
            this.section.data.color = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
            needUpdate = true
        }


        if (!this.section.data.list) {
            this.section.data.list = [];
            needUpdate = true;
        }


        if (needUpdate) {
            this.sectionCtrl.update(this.section);
        }

    },
    mounted() {
        console.log('tata :', this.$el.scrollTop, wwLib.getFrontDocument().body.scrollTop)
        // setInterval(() => {
        //     this.active = !this.active;
        // }, 4000);

        this.activeFeature = this.$refs[`content_${this.activeFeatureIndex}`]
        let lastScrollTop = window.pageYOffset || document.documentElement.scrollTop
        window.addEventListener('scroll', this.onScroll)
        console.log('ref', this.$refs['content_1'])
    },
    beforeDestroy() {
        window.removeEventListener('scroll', this.onScroll)
    },
    methods: {
        /* wwManager:start */
        add(list, options) {
            try {
                list.splice(options.index, 0, options.wwObject);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        remove(list, options) {
            try {
                list.splice(options.index, 1);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        /* wwManager:end */
        onScroll(event) {
            // console.log('event', event, window.innerHeight, this.$el.getBoundingClientRect().top)
            console.log(this.activeFeatureIndex)
            console.log(this.$refs[`content_${this.activeFeatureIndex}`].getBoundingClientRect().top, window.innerHeight / 2, this.onTheMove)
            let scrollTop = window.pageYOffset || document.documentElement.scrollTop;
            if (scrollTop > this.lastScrollTop) {
                // scrolling down
                if (Math.round(this.$refs[`content_${this.activeFeatureIndex}`].getBoundingClientRect().top) < Math.round(window.innerHeight / 2) && !this.onTheMove) {
                    this.onTheMove = true;
                    this.nextFeature()
                }

            } else {
                // scrolling up
                if (Math.round(this.$refs[`content_${this.activeFeatureIndex}`].getBoundingClientRect().top) > Math.round(window.innerHeight / 2) && !this.onTheMove) {
                    this.onTheMove = true;
                    this.previousFeature()
                }
            }
            this.lastScrollTop = scrollTop <= 0 ? 0 : scrollTop; // For Mobile or negative scrolling
            // if top pass the line at scroll down next
            // if top pass the line at scroll down up previous
        },
        nextFeature() {
            console.log('next')
            // First time
            if (this.activeFeatureIndex == 0) {
                this.features[this.activeFeatureIndex].active = true;
                this.activeFeatureIndex++
                this.onTheMove = false;
                return;
            }
            // After
            this.features[this.activeFeatureIndex].active = false;
            if (this.activeFeatureIndex == this.features.length - 1) return;



            setTimeout(() => {
                this.activeFeatureIndex++
                this.activeFeature = this.$refs[`content_${this.activeFeatureIndex}`]
                this.features[this.activeFeatureIndex].active = true;
                this.onTheMove = false;
            }, 600);
        },
        previousFeature() {
            console.log('previous')
            this.features[this.activeFeatureIndex].active = false;
            if (this.activeFeatureIndex == 0) {
                this.onTheMove = false;
                return;
            }
            this.activeFeatureIndex--;
            this.activeFeature = this.$refs[`content_${this.activeFeatureIndex}`]
            setTimeout(() => {
                this.features[this.activeFeatureIndex].active = true;
                this.onTheMove = false;
            }, 600);

        }
    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang="scss" scoped>
.scroll-fixed-features {
    padding: 100px 5%;
    pointer-events: all;
    .content {
        position: relative;
        .left-content {
            position: absolute;
            width: 40%;
            height: 100%;
            padding-right: 80px;
            .fixed-left {
                position: sticky;
                top: 50px;
                .feature {
                    margin-top: 20px;
                    height: 28px;
                    transition: height 0.6s;
                    .details {
                        transition: transform 0.6s, opacity 0.6s;
                        transform: translate3d(0px, -10px, 0px) scale3d(1, 1, 1)
                            rotateX(0deg) rotateY(0deg) rotateZ(0deg)
                            skew(0deg, 0deg);
                        transform-style: preserve-3d;
                        opacity: 0;
                        &.show {
                            transform: translate3d(0px, 0px, 0px)
                                scale3d(1, 1, 1) rotateX(0deg) rotateY(0deg)
                                rotateZ(0deg) skew(0deg, 0deg);
                            opacity: 1;
                        }
                    }
                    &.active {
                        height: 200px;
                    }
                }
            }
        }
        .right-content {
            width: 60%;
            margin-left: 40%;
        }
    }
}

.background {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
}
</style>
