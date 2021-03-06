<template>
	<nav class="navbar" v-bind:class="navbarClass">
		<div class="container-fluid">
			<div class="navbar-header">
				<slot name="sidebar-toggle-button"></slot>
				<slot name="brand"></slot>
				<button type="button" class="navbar-toggle" data-toggle="collapse" data-target="#collapse{{ id }}">
					<span class="sr-only">Toggle navigation</span>
					<span class="icon-bar"></span>
					<span class="icon-bar"></span>
					<span class="icon-bar"></span>
				</button>
			</div>
			<div class="navbar-collapse collapse" id="collapse{{ id }}">
				<slot></slot>
			</div>
		</div>
	</nav>
</template>

<script>
	try {
		require('jquery.breakpoints/breakpoints')
	}
	catch (e) {
		// do nothing
		// try catch required to avoid 'jQuery is not defined' error
		// when required in a node (non-browser) environment
	}
	import randString from 'mout/random/randString'

	export default {
		data () {
			return {
				id: 'navbar-' + randString()
			}
		},
		props: {
			skin: {
				type: String,
				required: false
			},
			fixed: {
				type: String,
				required: false
			},
			static: {
				type: String,
				required: false
			},
			absolute: {
				type: String,
				required: false
			},
			size: {
				type: String,
				required: false
			},
			inContent: {
				type: Boolean
			}
		},
		computed: {
			navbarClass () {
				var classes = {}
				if (this.skin) {
					classes['navbar-skin-' + this.skin] = true
				}
				else {
					classes['navbar-default'] = true
				}
				if (this.fixed) {
					classes['navbar-fixed-' + this.fixed] = true
				}
				if (this.static) {
					classes['navbar-static-' + this.static] = true
				}
				if (this.absolute) {
					classes['navbar-absolute-' + this.absolute] = true
				}
				if (this.size) {
					classes['navbar-size-' + this.size] = true
				}
				return classes
			}
		},
		methods: {
			layout () {
				return $(this.$el).parents('.la-co:first')
			},
			addInContentLayoutClass () {
				this.layout().addClass('navbar-content')
			},
			removeInContentLayoutClass () {
				this.layout().removeClass('navbar-content')
			},
			enableInContent () {
				this.addInContentLayoutClass()
			},
			inContentSlot () {
				return $(this.$el).is('[slot="navbar-content"]')
			},
			enableFixed () {
				if (this.fixed === 'top') {
					this.layout().addClass('ls-top-navbar')
				}
			},
			disableFixed () {
				this.layout().removeClass('ls-top-navbar')
			},
			sidebarTransitionsEnabled () {
				return this.layout().hasClass('st-la')
			},
			breakpoints (reset) {

				// @TODO: setBreakpoints overrides previous breakpoints
				// $(window).setBreakpoints({
				// 	breakpoints: ['320', '480']
				// })

				$(window)[reset ? 'off' : 'on']('enterBreakpoint320', function () {
					this.lastFixed = this.fixed
					this.fixed = 'top'
					if (!this.sidebarTransitionsEnabled() || !this.inContentSlot()) {
						this.lastInContent = this.inContent
						this.inContent = false
					}
				}.bind(this))

				// @TODO: shared breakpoints
				let breakpoints = [480, 768, 992, 1200, 1600]
				breakpoints.forEach(function (breakpoint) {
					$(window)[reset ? 'off' : 'on'](`enterBreakpoint${ breakpoint }`, function () {
						if (typeof this.lastFixed !== 'undefined') {
							this.fixed = this.lastFixed
						}
						if (typeof this.lastInContent !== 'undefined') {
							this.inContent = this.lastInContent
						}
					}.bind(this))
				}, this)
			}
		},
		ready () {
			if (this.inContent) {
				this.enableInContent()
			}
			if (this.fixed) {
				this.enableFixed()
			}
			if (this.inContentSlot() && !this.inContent) {
				this.inContent = true
			}
			this.breakpoints()
		},
		watch: {
			inContent (value) {
				if (value) {
					return this.enableInContent()
				}
				this.removeInContentLayoutClass()
			},
			fixed (value) {
				if (value) {
					return this.enableFixed()
				}
				this.disableFixed()
			}
		},
		beforeDestroy () {
			this.removeInContentLayoutClass()
			this.disableFixed()
			this.breakpoints(true)
		}
	}
</script>

<style lang="sass">
	@import "../sass/navbar/navbar";
	@import "../sass/layout/navbar";
</style>