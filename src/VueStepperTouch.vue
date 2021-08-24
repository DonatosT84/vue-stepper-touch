<script lang="ts">
import { defineComponent, ref, computed, h, onMounted, PropType } from 'vue';
import { gsap } from 'gsap';
import { Draggable } from 'gsap/Draggable';

gsap.registerPlugin(Draggable);

export default /*#__PURE__*/defineComponent({
  name: 'VueStepperTouch',
  props: {
    initialValue: {
      type: Number,
      default: 0,
    },
    min: {
      type: Number,
      default: 0,
    },
    max: {
      type: Number,
      default: 100
    },
    size: {
      type: Number,
      default: 3,
    },
    color: {
      type: String,
      default: '#6d72fe'
    },
    background: {
      type: String,
      default: '#fff'
    },
    opacity: {
      type: Number,
      default: 25
    },
    fontFamily: {
      type: String,
      default: 'Poppins'
    },
    axis: {
      type: String as PropType<Draggable.DraggableType>,
      enum: ['x', 'y'],
      default: 'x'
    }
  },
  setup(props) {
    // template refs
    const stepper = ref<HTMLElement | null>(null);
    const indicator = ref<HTMLElement | null>(null);
    const minus = ref<HTMLElement | null>(null);
    const plus = ref<HTMLElement | null>(null);

    // data
    const start = ref<number>(0);
    const end = ref<number>(0);
    const value = ref<number>(props.initialValue);
    const multiplier = ref<number>(2.3);
    const timeout = ref<NodeJS.Timeout | number | undefined>();
    const interval = ref<NodeJS.Timeout | number | undefined>();
    const counting = ref<boolean>(false);
    const isHorizontal = ref<boolean>(props.axis === 'x');
    const cssVars = ref<object | null>(null);

    // computed
    const classObject = computed<string>(() => {
      return isHorizontal.value ? 'horizontal' : 'vertical';
    })

    const fullSize = props.size * multiplier.value + 'rem';
    const stepperWidth = isHorizontal.value ? fullSize : props.size + 'rem';
    const stepperHeight = isHorizontal.value ? props.size + 'rem' : fullSize;

    cssVars.value = {
      '--size': props.size + 'rem',
      '--font-size': props.size * 0.6 + 'rem',
      '--font-family': props.fontFamily,
      '--color': props.color,
      '--background': props.background,
      '--background-alpha': convertHex(props.background, props.opacity),
      '--indicator-size': props.size - (props.size / 20) + 'rem',
      '--stepper-width': stepperWidth,
      '--stepper-height': stepperHeight
    }

    // methods
    function draggableHandler() {
      if (indicator.value && stepper.value) {
        Draggable.create(indicator.value, {
          type: props.axis,
          bounds: stepper.value,
          throwProps: true,
          edgeResistance: .95,
          dragResistance: .8,
          throwResistance: .9,
          snap: [0],
          ease: 'elastic.out(1, 0.75)',
          onRelease: function () {
            gsap.to(this.target, {
              x: this.startX,
              y: this.startY,
              ease: 'bounce.out'
            });
          },
          onDrag: function () {
            if (isHorizontal.value) {
              end.value = this.endX;
            } else {
              start.value = this.endY;
            }
          },
          onDragStart: function () {
            if (isHorizontal.value) {
              start.value = this.startX;
            } else {
              end.value = this.startY;
            }

            timeout.value = setTimeout(() => {
              counting.value = true;

              interval.value = setInterval(() => {
                count();
              }, 200);
            }, 500);
          },
          onDragEnd: function () {
            clearTimeout(<NodeJS.Timeout>timeout.value);
            clearInterval(<NodeJS.Timeout>interval.value);

            if (!counting.value) {
              count();
            }

            counting.value = false;
          }
        });
      }
    }

    function countDownHandler() {
      countDown();
      toggleMinusPlus();
    }

    function countUpHandler() {
      countUp();
      toggleMinusPlus();
    }

    function count() {
      let distance = end.value - start.value;

      if (distance > 0) {
        if (value.value + 1 <= props.max) {
          countUp();
        }
      } else if (distance < 0) {
        if (value.value - 1 >= props.min) {
          countDown();
        }
      }

      toggleMinusPlus();
    }

    function countDown() {
      if (value.value > props.min) value.value--;
    }

    function countUp() {
      if (value.value < props.max) value.value++;
    }

    function toggleMinusPlus() {
      if (minus.value) {
        if (value.value - 1 < props.min) {
          minus.value.classList.add('disabled');
        } else {
          minus.value.classList.remove('disabled');
        }
      }

      if (plus.value) {
        if (value.value + 1 > props.max) {
          plus.value.classList.add('disabled');
        } else {
          plus.value.classList.remove('disabled');
        }
      }
    }

    function convertHex(color : string, opacity : number) {
      color = color.replace('#', '')
      const r = parseInt(color.substring(0, 2), 16)
      const g = parseInt(color.substring(2, 4), 16)
      const b = parseInt(color.substring(4, 6), 16)
      return `rgba(${r}, ${g}, ${b}, ${opacity / 100})`
    }

    onMounted(() => {
      toggleMinusPlus();
      draggableHandler();
    });

    return () => h('div', { 'class': 'stepper', ref : stepper, style : cssVars.value}, [
        h('div', { 'class' : 'indicator', ref : indicator, style : cssVars.value}, [value.value]),
        h('span', { 'class' : `minus ${classObject.value}`, ref : minus, style : cssVars.value, onClick : countDownHandler }, '−'),
        h('span', { 'class' : `plus ${classObject.value}`, ref : plus, style : cssVars.value, onClick : countUpHandler }, '+'),
    ]);
  }
});
</script>

<style lang="scss">

$background: var(--background);
$background-alpha: var(--background-alpha);
$color: var(--color);
$font-family: var(--font-family);
$font-size: var(--font-size);
$size: var(--size);
$indicator-size: var(--indicator-size);
$stepper-width: var(--stepper-width);
$stepper-height: var(--stepper-height);

* {
  box-sizing: border-box;

&:before,
&:after {
   box-sizing: border-box;
 }
}

.stepper {
  width: $stepper-width !important;
  height: $stepper-height !important;
  border-radius: $size;
  background: $background-alpha;
  position: relative;
  font-family: $font-family;
  overflow: hidden;

  .indicator {
    background: $color;
    width: $indicator-size;
    height: $indicator-size;
    border-radius: 50%;
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    margin: auto;
    box-shadow: 0 0 1.25rem rgba(0, 0, 0, .2);
    color: $background;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: $font-size;
  }

  .minus,
  .plus {
    color: $color;
    font-size: $font-size;
    position: absolute;
    line-height: 1;
    cursor: pointer;

  &.disabled {
     opacity: .25;
   }
  }
}

.minus.horizontal,
.plus.horizontal {
  top: 50%;
  transform: translateY(-50%);
}

.minus.vertical,
.plus.vertical {
  left: 50%;
  transform: translateX(-50%);
}

.minus.horizontal {
  left: .2rem;
}

.minus.vertical {
  bottom: -0.05rem;
}

.plus.horizontal {
  right: .2rem;
}

.plus.vertical {
  top: -0.05rem;
}
</style>
