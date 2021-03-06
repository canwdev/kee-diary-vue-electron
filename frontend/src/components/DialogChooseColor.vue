<template>
  <q-dialog
      v-model="mVisible"
      transition-show="fade"
      transition-hide="fade"
  >
    <q-card>
      <q-card-section>
        <div class="text-h6 row items-center">
          <span class="q-ml-md">{{ $t('choose') }} {{
              isFgColor ? $t('foreground') : $t('background')
            }} {{ $t('color') }}</span>
          <q-space/>
          <q-toggle
              v-model="isFgColor"
              :color="toggleColor"
              keep-color
              left-label
          >
          </q-toggle>
        </div>
      </q-card-section>

      <q-separator/>

      <q-card-section style="max-height: 70vh" class="scroll">
        <q-card flat class="q-gutter-md">
          <ColorItem
              v-for="(item, i) in palette"
              :key="i"
              :color="item.color"
              :name="item.name"
              :is-active="selectedColor === item.color"
              @click.native="handleSelect(item.color)"
          />
        </q-card>
      </q-card-section>

      <q-separator/>

      <q-card-actions align="right">
        <q-btn flat :label="$t('cancel')" color="primary" v-close-popup/>
        <q-btn
            :disabled="selectedColor === color"
            @click="handleChoose"
            flat :label="$t('choose')" color="primary"
        />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script>
import {palette} from "@/utils/enum"
import ColorItem from '@/components/ColorItem'

function getTypeColor(isFgColor, item) {
  if (!item) return
  return isFgColor ? item.fgColor : item.bgColor
}

function getColorName(color) {
  const {name} = palette.filter(i => i.color === color)[0] || {}
  return name
}

export default {
  name: "DialogChooseIcon",
  components: {
    ColorItem
  },
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    item: {
      type: Object,
      default: null
    }
  },
  computed: {
    mVisible: {
      get() {
        return this.visible
      },
      set(nv) {
        this.$emit('update:visible', nv)
      }
    },
    color() {
      if (!this.item) {
        return null
      }
      return getTypeColor(this.isFgColor, this.item)
    },
    toggleColor() {
      return getColorName(getTypeColor(this.isFgColor, this.item))
    }
  },
  watch: {
    item: {
      handler(nv) {
        if (!nv) {
          return
        }
        this.selectedColor = getTypeColor(this.isFgColor, this.item)
      },
      immediate: true
    },
    isFgColor(nv) {
      this.selectedColor = getTypeColor(nv, this.item)
    }
  },
  data() {
    return {
      palette: Object.freeze(palette),
      selectedColor: null,
      isFgColor: true
    }
  },
  methods: {
    handleSelect(color) {
      if (this.selectedColor === color) {
        this.handleChoose()
      } else {
        this.selectedColor = color
      }
    },
    handleChoose() {
      this.$emit('onChoose', {
        type: this.isFgColor ? 'fgColor' : 'bgColor',
        value: this.selectedColor
      })
      this.mVisible = false
    }
  }
}
</script>

<style lang="stylus" scoped>

</style>
