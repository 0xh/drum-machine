<template>
  <svg width="34" height="50" class="knob" @mousedown="activate" @touchstart="activate">
    <circle class="shadow" cx="17" cy="17" r="16"></circle>
    <circle class="mainCircle" cx="17" cy="15" r="13" 
      :class="{ active: active}" />
    <line x1="17" y1="15" x2="17" y2="3" :transform="knobRotation"/>
    <text x="17" y="43">{{value | round}}</text>   
  </svg>  
</template>

<script>
export default {
  name: 'knob',
  props: ['max','min','value','step'],
  data () {
    return {
      internalValue: this.mapNumber(this.value, this.min, this.max, 0, 100),
      active: false,
      initialX: undefined,
      initialY: undefined,
      initialDragValue: undefined,
      shiftPressed: false
    }
  },
  created () {
    document.addEventListener('keydown', (e) => {
      if (e.key=='Shift') this.shiftPressed = true
    })
    document.addEventListener('keyup', (e) => {
      if (e.key=='Shift') this.shiftPressed = false
    })
  },
  watch: {
    value: function(newVal, oldVal){
      this.internalValue = this.mapNumber(newVal, this.min, this.max, 0, 100)
    }
  },
  filters: {
    round(val){
      return Math.floor(val*20)/20
    }
  },
  computed:{
    knobRotation(){
      let rotation = this.mapNumber(this.internalValue, 0,100,0,270)-135
      return `rotate( ${rotation} 17 15)`
    }
  },
  methods: {
    mapNumber(value,inputmin,inputmax,rangemin,rangemax){
      rangemax = parseFloat(rangemax)
      rangemin = parseFloat(rangemin)
      inputmax = parseFloat(inputmax)
      inputmin = parseFloat(inputmin)
      let result = (value- inputmin) * (rangemax - rangemin) / (inputmax - inputmin) + rangemin;
      return result
    },
    activate(event){
      this.initialX = event.pageX || event.changedTouches[0].pageX
      this.initialY = event.pageY || event.changedTouches[0].pageY
      this.active = true
      this.initialDragValue = this.internalValue
      document.onmouseup = this.deactivate
      document.addEventListener('touchend', this.deactivate)      
      document.onmousemove = this.dragHandler
      document.addEventListener('touchmove',this.dragHandler)
    },
    dragHandler(e){
      let xLocation = e.pageX || e.changedTouches[0].pageX
      let yLocation = e.pageY || e.changedTouches[0].pageY
      if (Math.abs(xLocation - this.initialX)> Math.abs(yLocation - this.initialY))
        {
          if(this.shiftPressed){
            this.internalValue = this.initialDragValue + (xLocation - this.initialX)/10
          } else {
            this.internalValue = this.initialDragValue + (xLocation - this.initialX)/3
          }
        } else {
          if(this.shiftPressed){
            this.internalValue = this.initialDragValue + (this.initialY-yLocation)/10
          } else {
            this.internalValue = this.initialDragValue + (this.initialY - yLocation)/3
          }
        }
      if (this.internalValue>100) this.internalValue = 100
      if (this.internalValue<0) this.internalValue = 0
      if(isNaN(this.internalValue)) this.internalValue = this.initialDragValue
      this.$emit('input', this.mapNumber(this.internalValue, 0,100,this.min,this.max))
    },
    deactivate(){
      document.onmouseup = undefined
      document.onmousemove = undefined
      document.removeEventListener('touchmove',this.dragHandler)
      document.removeEventListener('touchend',this.deactivate)
      this.active = false
    }
  }
}
</script>

<style scoped>
.knob{
  user-select: none;
  margin-left: 10px;
}
circle.mainCircle{
  fill: #CDD8B7;
  stroke: #A2AB8F;
  stroke-width: 3px;
}

circle.mainCircle.active{
  fill :#B4BE9F;
}

circle.shadow{
  fill: #CDCCCC;

}

line {
  stroke: #666B5B;
  stroke-width: 3px;
}

text {
  font-size: 8px;
  text-anchor: middle;
  fill: #777;
}
</style>

