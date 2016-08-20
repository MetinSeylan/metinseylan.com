<template>
    <div></div>
</template>

<script>
    import eve from 'eve';
    window.eve = eve;
    import Raphael from 'raphael'
    window.Raphael = Raphael;

    import 'justgage'



    export default{
        props: [
            'max',
            'value',
                'title'
        ],
        data(){
            return {
                gage: null
            }
        },
        ready(){
            this.run();
        },
        methods: {
            run(){
                this.gage = new JustGage({
                    refreshAnimationTime: "10",
                    refreshAnimationType: "linear",
                    hideInnerShadow: true,
                    parentNode: this.$el,
                    value: 0,
                    min: this.value,
                    max: this.max,
                    title: this.title
                });

            }
        },
        watch: {
            value(value){
                this.gage.refresh(value);
            }
        }
    }
</script>
