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
                <div class="fixed-left-container">
                    <div class="fixed-left">
                        <div v-for="(feature, index) in features" :key="index" class="feature" :class="{'active': feature.active}">
                            <div class="title">
                                <div v-if="editMode" class="contextmenu-container">
                                    <div class="index">{{index + 1}}</div>
                                    <wwContextMenu class="contextmenu" tag="div" @ww-add-before="addFeature(index, 'before')" @ww-add-after="addFeature(index, 'after')" @ww-remove="removeFeature(index)">
                                        <div class="wwi wwi-config"></div>
                                    </wwContextMenu>
                                </div>
                                <div v-if="editMode" class="toggle-active">
                                    <label>
                                        <input type="checkbox" v-model="feature.active" />
                                        <span class="check"></span>
                                    </label>
                                </div>
                                <wwObject v-bind:ww-object="feature.title"></wwObject>
                            </div>
                            <div class="details" :class="{'show': feature.active}">
                                <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="feature.details" class="list" @ww-add="add(feature.details, $event)" @ww-remove="remove(feature.details, $event)">
                                    <wwObject tag="div" v-for="object in feature.details" :key="object.uniqueId" :ww-object="object"></wwObject>
                                </wwLayoutColumn>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="fixed-right-container">
                    <div class="fixed-right">
                        <wwObject v-bind:ww-object="features[activeFeatureIndex].rightBackground"></wwObject>
                    </div>
                </div>
            </div>
            <div ref="rightContent" class="right-content">
                <div class="content-container" :ref="`content_${index}`" v-for="(feature, index) in section.data.features" :key="index">
                    <div v-if="editMode" class="contextmenu-container">
                        <div class="index">{{index+1}}</div>
                    </div>
                    <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="feature.contents" class="list" @ww-add="add(feature.contents, $event)" @ww-remove="remove(feature.contents, $event)">
                        <wwObject tag="div" v-for="object in feature.contents" :key="object.uniqueId" :ww-object="object"></wwObject>
                    </wwLayoutColumn>
                    <!-- <div ref="content_0" :style="{height:'400px', width: '100%', 'background-color': 'blue', 'margin-bottom':'300px'}"></div> -->
                </div>
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
            test: false,
            features: [
                {
                    active: false,
                    title: null,
                    details: [],
                    contents: [],
                    rightBackground: {}
                }
            ],
            lastScrollTop: 0,
            onTheMove: false
        }
    },
    computed: {
        section() {
            return this.sectionCtrl.get();
        },
        editMode() {
            return this.sectionCtrl.getEditMode() == 'CONTENT'
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
        this.features = this.section.data.features;
        this.features[0].rightBackground = wwLib.wwObject.getDefault({
            type: 'ww-image'
        })
        setTimeout(() => {
            console.log(this.section.data.features)
            this.section.data.features[0].active = true;
        }, 5000);
        if (!this.section.data.features[0].title) {
            this.section.data.features[0].title = wwLib.wwObject.getDefault({
                type: 'ww-text',
                data: {
                    text: {
                        fr_FR: 'Hello World FR !',
                        en_GB: 'Hello World !'
                    }
                }

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
        this.activeFeature = this.$refs[`content_${this.activeFeatureIndex}`]
        let lastScrollTop = window.pageYOffset || document.documentElement.scrollTop
        window.addEventListener('scroll', this.onScroll)
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
        addFeature(_index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                const newFeature = this.getNewFeature()
                this.section.data.features.splice(index, 0, newFeature);
                this.sectionCtrl.update(this.section);
                this.restartScrollListerner()
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        getNewFeature() {
            const newFeature = JSON.parse(JSON.stringify(this.section.data.features[0]))
            wwLib.wwUtils.changeUniqueIds(newFeature)
            return newFeature
        },
        removeFeature(index) {
            try {
                this.section.data.features.splice(index, 1);
                if (!this.section.data.features.length) {
                    this.addFeature(0, 'after');
                }
                this.sectionCtrl.update(this.section);
                this.restartScrollListerner()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        /* wwManager:end */
        onScroll(event) {
            let scrollTop = window.pageYOffset || document.documentElement.scrollTop;

            if (scrollTop > this.lastScrollTop) {
                // scrolling down
                if (this.activeFeatureIndex == this.features.length - 1) return;

                let featureWatched = this.$refs[`content_${(this.activeFeatureIndex + 1)}`]
                if (Math.round(featureWatched.getBoundingClientRect().top) < Math.round(window.innerHeight / 2)) {
                    this.nextFeature('next')
                }

            } else {
                // scrolling up
                if (this.activeFeatureIndex == 0) return;

                let featureWatched = this.$refs[`content_${(this.activeFeatureIndex)}`]
                if (Math.round(featureWatched.getBoundingClientRect().top) > Math.round(window.innerHeight / 2) && !this.onTheMove) {
                    this.onTheMove = true;
                    this.previousFeature('previous')
                }
            }
            this.lastScrollTop = scrollTop <= 0 ? 0 : scrollTop; // For Mobile or negative scrolling
        },
        restartScrollListerner() {
            window.removeEventListener('scroll', this.onScroll)
            window.addEventListener('scroll', this.onScroll)
        },
        nextFeature(direction) {
            this.features[this.activeFeatureIndex].active = false;
            if (direction == 'next') {
                this.activeFeatureIndex++;
            } else {
                this.activeFeatureIndex--;
            }
            this.restartScrollListerner()
            clearTimeout(this.activeTimout)
            this.activeTimout = setTimeout(() => {
                this.activeFeature = this.$refs[`content_${this.activeFeatureIndex}`];
                this.features[this.activeFeatureIndex].active = true;
            }, 600);
        },
        previousFeature() {

            this.features[this.activeFeatureIndex].active = false;
            setTimeout(() => {
                this.activeFeatureIndex--;
                this.activeFeature = this.$refs[`content_${this.activeFeatureIndex}`];
                this.features[this.activeFeatureIndex].active = true;
                this.onTheMove = false;
                this.restartScrollListerner();
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
    position: relative;
    pointer-events: all;
    min-height: 100vh;
    .content {
        position: relative;
        .left-content {
            position: absolute;
            height: 100vh;
            width: 100%;
            .fixed-left-container {
                position: absolute;
                width: 35%;
                height: 100%;
                margin-left: 5%;
                top: 0;
                left: 0;
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
                            transform: translate3d(0px, -10px, 0px)
                                scale3d(1, 1, 1) rotateX(0deg) rotateY(0deg)
                                rotateZ(0deg) skew(0deg, 0deg);
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
            .fixed-right-container {
                position: absolute;
                width: 60%;
                height: 100%;
                top: 0;
                right: 0;
                .fixed-right {
                    position: sticky;
                    top: 50px;
                    width: 100%;
                    right: 0;
                }
            }
            .title {
                position: relative;
            }
        }
        .right-background {
            z-index: 1;
            position: absolute;
        }
        .right-content {
            z-index: 2;
            position: relative;
            width: 55%;
            margin-left: 40%;
            .content-container {
                position: relative;
            }
        }
        .contextmenu-container {
            position: absolute;
            top: 0;
            left: 0;
            transform: translateY(-100%);
            border-top-left-radius: 20px;
            border-top-right-radius: 20px;
            padding: 5px;
            text-align: center;
            background-color: #ef811a;
            color: white;
            .index {
            }
            .contextmenu {
                font-size: 1.2rem;
                cursor: pointer;
                z-index: 2;
            }
        }
        .toggle-active {
            position: absolute;
            top: 0;
            left: 30px;
            transform: translateY(-100%);
            input[type="checkbox"] {
                -webkit-appearance: none;
                appearance: none;
                visibility: hidden;
                display: none;
            }

            .check {
                position: relative;
                display: block;
                width: 40px;
                height: 18px;
                background-color: #ffffff;
                border: 1px solid #19947c;
                cursor: pointer;
                border-radius: 20px;
                overflow: hidden;
                transition: ease-in 0.5s;
            }

            input:checked[type="checkbox"] ~ .check {
                background-color: #19947c;
                /*   box-shadow: 0 0 0 1200px #092c3e; */
            }

            .check:before {
                content: "";
                position: absolute;
                top: 3px;
                left: 4px;
                background-color: #19947c;
                width: 11px;
                height: 11px;
                border-radius: 50%;
                transition: all 0.5s;
            }

            input:checked[type="checkbox"] ~ .check:before {
                // transform: translateX(50px);
                left: calc(100% - 15px);
                background-color: #ffffff;
            }
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
