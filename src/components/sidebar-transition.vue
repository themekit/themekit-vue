<script>
	import Sidebar from './sidebar.vue'
	import camelCase from 'mout/string/camelCase'

	export default {
		name: 'sidebar-transition',
		mixins: [ Sidebar ],
		data () {
			return {
				sidebarTransitions: false,
				animating: false,
				duration: 500
			}
		},
		props: {
			effect: {
				type: String,
				validator (value) {
					let effects = [
						'reveal',
						'slide-in',
						'slide-along',
						'slide-out-reverse',
						'scale-up',
						'fall-down',
						'push',
						'push-3d-rotate-in',
						'push-3d-rotate-out',
						'push-3d-rotate-delay'
					]
					return effects.indexOf(value) !== -1
				}
			},
			overlay: {
				type: Boolean
			}
		},
		computed: {
			sidebarClass () {
				var classes = this.sidebarBaseClass
				if (this.sidebarTransitionsEnabled()) {
					classes[this.sidebarEffectClass] = true
				}
				return classes
			},
			layoutSidebarTransitionClasses () {
				var classes = []
				if (this.size) {
					classes.push(this.layoutSidebarTransitionClass(this.direction, this.size))
				}
				this.screens.forEach(function (screen) {
					let sizeProperty = camelCase('size-' + screen)
					if (this[sizeProperty]) {
						classes.push(this.layoutSidebarTransitionClass(this.direction, this[sizeProperty], screen))
					}
				}, this)
				return classes.join(' ')
			},
			sidebarEffectClass () {
				return 'st-' + this.effect
			},
			toggleLayoutClasses () {
				if (this.toggleLayout !== 'auto') {
					return this.toggleLayout.split(',').join(' ')
				}
				var match = new RegExp('si-' + this.direction + '(\\S+)', 'ig')
				var layoutClasses = this.layout().get(0).className.match(match)
				if (layoutClasses !== null && layoutClasses.length) {
					return layoutClasses.join(' ')
				}
				return false
			}
		},
		methods: {
			container () {
				return this.layout().find('.st-co:first')
			},
			sidebarTransitionsEnabled () {
				if (!this.effect) {
					return false
				}
				return this.layout().hasClass('st-la')
			},
			layoutSidebarTransitionClass (direction, size, screen) {
				var className = 'st-effect-'
				className += direction
				className += size
				if (screen) {
					className += '-' + screen
				}
				return className
			},
			isAnimating () {
				if (!this.sidebarTransitionsEnabled()) {
					return false
				}
				return this.animating
			},
			onOpen () {
				if (this.sidebarTransitionsEnabled()) {
					this.emit('show')
					this.addLayoutClasses()
					this.onEnter()
				}
				else {
					Sidebar.methods.onOpen.call(this)
				}
			},
			onClose () {
				if (this.sidebarTransitionsEnabled()) {
					this.emit('hide')
					this.onLeave()
				} 
				else {
					Sidebar.methods.onClose.call(this)
				}
			},
			onEnter () {
				clearTimeout(this.leaveTimer)
				this.layout().addClass(this.layoutSidebarTransitionClasses)
				if (this.toggleLayout) {
					this.layout().addClass(this.toggleLayoutClasses)
				}
				this.sidebar().addClass(this.sidebarEffectClass)
				this.container().addClass(this.sidebarEffectClass)
				if (this.overlay) {
					this.container().addClass('st-pu-overlay')
				}
				this.isVisible = true
				this.enterTimer = setTimeout(function () {
					this.container().addClass('st-menu-open')
					this.sidebar().find('.simplebar').simplebar('recalculate')
				}.bind(this), 10)

				this.animating = false
				this.animatingTimer = setTimeout(function () {
					this.animating = false
					this.emit('change')
					this.emit('shown')
				}.bind(this), this.duration)

				this.doneTimer = setTimeout(function () {
					this.container().addClass('st-done')
				}.bind(this), this.duration + 140)
			},
			onLeave () {
				clearTimeout(this.enterTimer)
				clearTimeout(this.animatingTimer)
				clearTimeout(this.doneTimer)
				this.container().removeClass('st-menu-open st-done')
				this.animating = true
				this.leaveTimer = setTimeout(function () {
					this.removeLayoutClasses()
					this.container().get(0).className = 'st-co'
					this.layout().removeClass(this.layoutSidebarTransitionClasses)
					if (this.toggleLayout) {
						this.layout().removeClass(this.toggleLayoutClasses)
					}
					this.isVisible = false
					this.animating = false
					this.emit('change')
					this.emit('hidden')
				}.bind(this), this.duration)
			}
		},
		beforeDestroy () {
			clearTimeout(this.enterTimer)
			clearTimeout(this.leaveTimer)
			clearTimeout(this.animatingTimer)
			clearTimeout(this.doneTimer)
			this.removeLayoutClasses()
			this.layout().removeClass(this.layoutSidebarTransitionClasses)
			if (this.toggleLayout) {
				this.layout().removeClass(this.toggleLayoutClasses)
			}
		},
		watch: {
			direction (newValue, oldValue) {
				if (oldValue) {
					this.layout().removeClass(this.layoutSidebarTransitionClass(oldValue, this.size))
				}
				if (this.show) {
					this.layout().addClass(this.layoutSidebarTransitionClasses)
				}
			},
			size (newValue, oldValue) {
				if (oldValue) {
					this.layout().removeClass(this.layoutSidebarTransitionClass(this.direction, oldValue))
				}
				if (this.show) {
					this.layout().addClass(this.layoutSidebarTransitionClasses)
				}
			}
		}
	}
</script>

<style lang="sass">
	@import "../sass/sidebar/transition";
</style>