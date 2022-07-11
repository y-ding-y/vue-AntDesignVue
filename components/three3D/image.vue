<template>
    <div :class="'animated '+ animated" :style="styleSet"> 
    </div>
</template>

<script>
    //   import merge from 'deepmerge' 
    export default {
        props: {
            ss: {
                type: Object, default: () => ({
                    left: 0,
                    width: 100,
                    height: 100,
                    url: ""
                })
            },
            curdiv: { type: Boolean, default: false }
        },
        data() {
            return {
                animated: this.ss.inanimate,
                isshow: this.ss.isshow,
            }
        },
        watch: {
            isshow(vnew, vold) {
                if (this.isshow) {
                    this.animated = this.ss.inanimate
                }
                else if (vnew != vold && !this.isshow) {
                    this.animated = this.ss.outanimate
                }
            },

        },
        created() {
            this.isshow = this.ss.isshow
            if (this.ss.isshow) {
                this.animated = this.ss.inanimate
            }
            else {
                this.animated = this.ss.outanimate
            }
        },

        computed: {
            styleSet: function () {
                var style = {
                    border: this.curdiv ? '5px solid blue' : '5px',
                    position: 'absolute',
                    top: this.ss.top + 'px',
                    left: this.ss.left + 'px',
                    width: this.ss.width + "px",
                    height: this.ss.height + "px",
                    fontSize: '20px',
                    background: 'url(' + this.ss.url + ') no-repeat center / 100% 100%',
                    // backgroundColor: this.ss.backgroundColor,
                    zIndex: this.ss.zindex
                    //transform:this.ss.transform
                }
                // return merge(this.ss, style)
                return style

            },
        },
    }
</script>
