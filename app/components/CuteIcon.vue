<script lang="ts">
import { defineComponent, h, type PropType } from 'vue'

type IconPath = {
  d: string
  fill?: string
  stroke?: string
  strokeWidth?: number
  strokeLinecap?: 'round' | 'butt' | 'square'
  strokeLinejoin?: 'round' | 'miter' | 'bevel'
}

type IconRect = {
  x: number
  y: number
  width: number
  height: number
  rx?: number
  fill: string
}

type IconCircle = {
  cx: number
  cy: number
  r: number
  fill: string
}

type IconDefinition = {
  bg: string
  paths?: IconPath[]
  rects?: IconRect[]
  circles?: IconCircle[]
}

const ICONS: Record<string, IconDefinition> = {
  chevronLeft: {
    bg: '#FFE4E6',
    paths: [{ d: 'M13.8 7.5L9.3 12L13.8 16.5', stroke: '#9F1239', strokeWidth: 2, strokeLinecap: 'round', strokeLinejoin: 'round' }]
  },
  chevronRight: {
    bg: '#FFE4E6',
    paths: [{ d: 'M10.2 7.5L14.7 12L10.2 16.5', stroke: '#9F1239', strokeWidth: 2, strokeLinecap: 'round', strokeLinejoin: 'round' }]
  },
  close: {
    bg: '#FEE2E2',
    paths: [{ d: 'M8 8L16 16M16 8L8 16', stroke: '#991B1B', strokeWidth: 2, strokeLinecap: 'round' }]
  },
  gym: {
    bg: '#DCFCE7',
    rects: [
      { x: 5.2, y: 10.3, width: 3.2, height: 3.4, rx: 0.8, fill: '#166534' },
      { x: 15.6, y: 10.3, width: 3.2, height: 3.4, rx: 0.8, fill: '#166534' },
      { x: 8, y: 11, width: 8, height: 2, rx: 1, fill: '#22C55E' }
    ]
  },
  cardio: {
    bg: '#E0F2FE',
    paths: [{ d: 'M12 17.2C8.3 15.1 6.4 13.3 6.4 10.7C6.4 9.1 7.6 7.8 9.2 7.8C10.3 7.8 11.3 8.3 12 9.2C12.7 8.3 13.7 7.8 14.8 7.8C16.4 7.8 17.6 9.1 17.6 10.7C17.6 13.3 15.7 15.1 12 17.2Z', fill: '#0284C7' }]
  },
  rest: {
    bg: '#FEF3C7',
    rects: [
      { x: 5.8, y: 11.3, width: 12.4, height: 5.6, rx: 1.4, fill: '#B45309' },
      { x: 6.7, y: 8.1, width: 4.2, height: 3.3, rx: 0.9, fill: '#FDE68A' }
    ]
  },
  legs: {
    bg: '#EDE9FE',
    paths: [
      { d: 'M9.2 7.2L11.3 12.2L9.6 16.8', stroke: '#6D28D9', strokeWidth: 1.9, strokeLinecap: 'round', strokeLinejoin: 'round' },
      { d: 'M14.8 7.2L12.7 12.2L14.4 16.8', stroke: '#6D28D9', strokeWidth: 1.9, strokeLinecap: 'round', strokeLinejoin: 'round' }
    ]
  },
  ass: {
    bg: '#FCE7F3',
    paths: [{ d: 'M8.2 14.3C8.2 12.1 9.8 10.3 12 10.3C14.2 10.3 15.8 12.1 15.8 14.3', stroke: '#BE185D', strokeWidth: 2, strokeLinecap: 'round' }],
    circles: [
      { cx: 10.6, cy: 13.3, r: 1.2, fill: '#EC4899' },
      { cx: 13.4, cy: 13.3, r: 1.2, fill: '#EC4899' }
    ]
  },
  shoulders: {
    bg: '#E0E7FF',
    circles: [
      { cx: 9, cy: 10, r: 2, fill: '#4F46E5' },
      { cx: 15, cy: 10, r: 2, fill: '#4F46E5' }
    ],
    rects: [{ x: 9, y: 11.2, width: 6, height: 4.8, rx: 2.4, fill: '#6366F1' }]
  },
  chest: {
    bg: '#FFE4E6',
    paths: [{ d: 'M12 16.8C8.8 14.9 7.2 13.4 7.2 11.3C7.2 10 8.2 9 9.5 9C10.5 9 11.3 9.4 12 10.1C12.7 9.4 13.5 9 14.5 9C15.8 9 16.8 10 16.8 11.3C16.8 13.4 15.2 14.9 12 16.8Z', fill: '#E11D48' }]
  },
  back: {
    bg: '#D1FAE5',
    rects: [{ x: 8.2, y: 7.7, width: 7.6, height: 8.8, rx: 3.2, fill: '#059669' }],
    paths: [{ d: 'M12 9.2V15.2', stroke: '#ECFDF5', strokeWidth: 1.4, strokeLinecap: 'round' }]
  },
  biceps: {
    bg: '#FEF3C7',
    paths: [{ d: 'M8 14.5C9.2 12.4 11.2 11.3 13.4 11.3C14.8 11.3 16 12.4 16 13.8C16 15.4 14.7 16.7 13.1 16.7H10.1C8.9 16.7 8 15.7 8 14.5Z', fill: '#D97706' }],
    circles: [{ cx: 11.2, cy: 13.4, r: 1, fill: '#FDE68A' }]
  },
  triceps: {
    bg: '#DBEAFE',
    paths: [
      { d: 'M9.1 8.6L15.8 15.3', stroke: '#1D4ED8', strokeWidth: 2, strokeLinecap: 'round' },
      { d: 'M8.6 14.9C10.2 16.5 12.6 16.5 14.2 14.9', stroke: '#2563EB', strokeWidth: 1.7, strokeLinecap: 'round' }
    ]
  },
  fallback: {
    bg: '#E5E7EB',
    circles: [
      { cx: 9, cy: 10, r: 1, fill: '#6B7280' },
      { cx: 15, cy: 10, r: 1, fill: '#6B7280' }
    ],
    paths: [{ d: 'M8.6 14.5C9.4 15.2 10.6 15.6 12 15.6C13.4 15.6 14.6 15.2 15.4 14.5', stroke: '#6B7280', strokeWidth: 1.6, strokeLinecap: 'round' }]
  }
}

export default defineComponent({
  name: 'CuteIcon',
  props: {
    name: {
      type: String as PropType<string>,
      required: true
    },
    class: {
      type: String as PropType<string>,
      default: ''
    }
  },
  setup(props) {
    return () => {
      const icon: IconDefinition = Object.prototype.hasOwnProperty.call(ICONS, props.name)
        ? ICONS[props.name]!
        : ICONS.fallback!

      const children = [h('circle', { cx: 12, cy: 12, r: 10, fill: icon.bg })]

      if (icon.rects) {
        icon.rects.forEach((rect) => {
          children.push(h('rect', { ...rect }))
        })
      }

      if (icon.circles) {
        icon.circles.forEach((circle) => {
          children.push(h('circle', { ...circle }))
        })
      }

      if (icon.paths) {
        icon.paths.forEach((path) => {
          children.push(h('path', {
            d: path.d,
            fill: path.fill,
            stroke: path.stroke,
            strokeWidth: path.strokeWidth,
            strokeLinecap: path.strokeLinecap,
            strokeLinejoin: path.strokeLinejoin
          }))
        })
      }

      return h('svg', {
        viewBox: '0 0 24 24',
        fill: 'none',
        xmlns: 'http://www.w3.org/2000/svg',
        class: ['inline-block', props.class],
        ariaHidden: 'true'
      }, children)
    }
  }
})
</script>
