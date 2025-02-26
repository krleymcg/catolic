/*!
  ImageLinks - jQuery Interactive Image Map
  @name jquery.imagelinks.js
  @description jQuery plugin for creating an interactive image maps for news, posters, albums and etc
  @version 1.0.1
  @author Max Lawrence
  @site http://www.avirtum.com
*/

/* Available callbacks
  Callback Name         Description
  onPreInit(config)     called before the imagelinks is loaded & init, but the config is ready
  onLoad()              called before firing the ready event, all structures are ready
*/

/* Available events
  Plugin events are sent to notify code of interesting things that have taken place.
  Event Name                Handler
  imagelinks:ready          function(e, plugin)
*/

/* Changelog
  - 1.0.1 /26.03.2019/
  - Fix: bodyclick event marker handler for FireFox, IE
  
  - 1.0.0 /04.11.2018/
  - Initial release
*/

;(function($, window, document, undefined) {
	'use strict';
	
	//=============================================
	// Utils
	//=============================================
	function Util() {
		this.isMobile = function() {
			return /Mobi|Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
		};
		
		this.transitionEvent = function() {
			var el = document.createElement('fakeelement');
			
			var transitions = {
				'transition': 'transitionend',
				'OTransition': 'otransitionend',
				'MozTransition': 'transitionend',
				'WebkitTransition': 'webkitTransitionEnd'
			};
			
			for(var i in transitions){
				if (el.style[i] !== undefined){
					return transitions[i];
				}
			}
			
			return null;
		};
		
		this.animationEvent = function() {
			var el = document.createElement('fakeelement');
			
			var animations = {
				'animation'      : 'animationend',
				'MSAnimationEnd' : 'msAnimationEnd',
				'OAnimation'     : 'oAnimationEnd',
				'MozAnimation'   : 'mozAnimationEnd',
				'WebkitAnimation': 'webkitAnimationEnd'
			}
			
			for (var i in animations){
				if (el.style[i] !== undefined){
					return animations[i];
				}
			}
			
			return null;
		};
	};
	
	//=============================================
	// Data
	//=============================================
	var ITEM_DATA_INSTANCE = 'imagelinks-instance',
	ITEM_DATA_ID = 'imagelinks-id',
	INSTANCE_COUNTER = 0;
	
	function ImageLinks($container, config) {
		INSTANCE_COUNTER++;
		
		this.util = new Util();
		this.$container = null;
		this.config = null;
		this.controls = {};
		this.imageWidth = 0; // original image width
		this.imageHeight = 0; // original image height
		this.markers = [];
		this.tooltipZindex = [];
		this.bodyClick = false;
		this.themeClass = null;
		this.timerIdResize = null;
		this.timerIdScroll = null;
		this.ready = false;
		
		this._init($container, config);
	}
	ImageLinks.prototype = {
		VERSION: '1.0.0',
		
		//=============================================
		// Properties & methods (shared for all instances)
		//=============================================
		defaults: {
			theme: 'default',// theme name, you can create your own (see imagelinks.css file, the theme section)
			width: null, // the width of the container  (can be any of valid css units)
			height: null, // the height of the container (can be any of valid css units)
			imgSrc: null, // source for the background image
			delayResize: 100, // default time (ms) for onResize event timeout
			markers: [], // array of markers
			className: null, // additional css classes for the container
			onPreInit: null, // callback function(config) {...} fires before the imagelinks is loaded, but the config is ready
			onLoad: null, // callback function() {...} fires after the imagelinks is loaded
		},
		
		defaultsMarker: {
			x: 0, // x position of the marker in %
			y: 0, // y position  of the marker in %
			width: null, // the width of the marker (can be any of valid css units)
			height: null, // the height of the marker (can be any of valid css units)
			angle: null, // defines an angle (deg) that rotates the marker around a fixed point
			imgSrc: null, // source for the marker image
			link: null, // if set, the marker is a link
			linkNewWindow: false, // if set, open link in new window
			responsive: false, // sets the marker scaling depending on the size of the container
			noevents: false, // the marker is never the target of mouse events
			tooltip: null, // specify the config for the tooltip
			data: null, // the content of the marker
			dataSelector: null, // we will use this selector to fill the content of the marker
			className: null, // additional css classes for the marker which may enable additional styling for example (assigned to imgl-marker-wrap)
		},
		
		defaultsTooltip: {
			active: true, // the active state of the tooltip
			placement: 'top', // preferred position of the tooltip
			offset: {x:0, y:0}, // adjust the x & y offset of the tooltip.
			width: null, // the width of the tooltip (can be any of valid css unit)
			widthFromCSS: false, // the width of the tooltip will be taken from css
			trigger: 'hover', // the event which causes the tooltip to show, can be: hover, click, clickbody, focus, sticky (will be shown on load)
			followCursor: false, // determines if the tooltip follows the user's mouse cursor while showing (only for the hover trigger)
			responsive: false, // sets the tooltip scaling depending on the size of the container
			interactive: true, // determines if the tooltip should be interactive, i.e. able to be hovered over or clicked without hiding
			smart: false, // determines if the tooltip is placed within the viewport as best it can be if there is not enough space
			showOnInit: false, // automatically show tooltip on load (stickies will be shown anyway)
			showAnimation: null, // the type of show transition animation
			hideAnimation: null, // the type of hide transition animation
			duration: 300, // define animation duration in ms
			data: null, // the content of the tooltip
			dataSelector: null, // we will use this selector to fill the content of the tooltip
			className: null, // additional css classes for the tooltip which may enable additional styling for example
		},
		
		util: null,
		
		//=============================================
		// Private Methods
		//=============================================
		_init: function($container, config) {
			this.$container = $container;
			this.config = config;
			
			this._create();
		},
		
		_create: function() {
			var _self = this;
			function init() {
				_self._buildDOM();
				_self._setTheme(_self.config.theme);
				_self._bind();
				
				_self._initMarkers();
				_self._beforeReady();
				_self._updateSize();
				_self._ready();
			}
			
			var image = new Image();
			image.onload = $.proxy(function(xhr) {
				_self.imageWidth = image.width;
				_self.imageHeight = image.height;
				
				setTimeout(init, 300); // wait some time
			}, this);
			image.onerror = $.proxy(function(xhr) {
				console.error('Cannot load image "' + _self.config.imgSrc + '"');
			}, this);
			image.src = _self.config.imgSrc;
		},
		
		_setTheme: function(theme) {
			this.$container.removeClass(this.themeClass);
			
			this.themeClass = (theme ? 'imgl-theme-' + theme.toLowerCase() : null);
			
			this.$container.addClass(this.themeClass);
		},
		
		_buildDOM: function() {
			this.controls.$image = $('<div>').addClass('imgl-image').css({'background-image':'url(' + this.config.imgSrc + ')'});
			this.controls.$stage = $('<div>').addClass('imgl-stage');
			this.controls.$markers = $('<ul>').addClass('imgl-markers'); //!!!
			this.controls.$tooltips = $('<div>').addClass('imgl-tooltips');
			this.controls.$lastMarker = $('<li>').addClass('imgl-marker-last').attr('tabindex',1);
			
			this.controls.$stage.append(this.controls.$markers.append(this.controls.$lastMarker), this.controls.$tooltips);
			this.controls.$image.append(this.controls.$stage);
			
			this.$container.prepend(this.controls.$image).addClass('imgl-map').addClass(this.config.className);
			
			if(this.config.width!=null) {
				this.$container.css({'width': this.config.width});
			}
			
			if(this.config.height!=null) {
				this.$container.css({'height': this.config.height});
			}
		},
		
		_bind: function() {
			$(window).on('resize', $.proxy(this._onResize, this));
			$(window).on('scroll', $.proxy(this._onScroll, this));
			$('body').on('click', $.proxy(this._onBodyClickTooltip, this));
			
			this.controls.$lastMarker.on('focus', $.proxy(this._onLastMarkerFocus, this));
		},
		
		_unbind: function() {
			$(window).off('resize', $.proxy(this._onResize, this));
			$(window).off('scroll', $.proxy(this._onScroll, this));
			$('body').off('click', $.proxy(this._onBodyClickTooltip, this));
			
			this.controls.$lastMarker.off('focus', $.proxy(this._onLastMarkerFocus, this));
		},
		
		_beforeReady: function() {
			this.$container.addClass('imgl-before-ready');
			this.$container.get(0).offsetHeight;
			
			if(this.$container.is(':hidden')) {
				this.$container.css('display','block');
			}
			
			for(var i=0;i<this.markers.length;i++) {
				var marker = this.markers[i];
				
				// prepare tooltips
				if(!marker.cfg.tooltip.widthFromCSS) {
					var rect = marker.$tooltip.get(0).getBoundingClientRect(),
					width = (marker.cfg.tooltip.width ? marker.cfg.tooltip.width : rect.width + 1);
					marker.$tooltip.css({'width':width});
				}
				marker.$tooltipPos.css({'width':'auto'});
				
				if(marker.cfg.tooltip.showOnInit || marker.cfg.tooltip.trigger == 'sticky') {
					this._showTooltip(marker);
				}
			}
			
			this.$container.removeClass('imgl-before-ready');
		},
		
		_ready: function() {
			if(this.config.onLoad) {
				var fn = null;
				if(typeof this.config.onLoad == 'string') {
					try {
						fn = new Function(this.config.onLoad);
					} catch(ex) {
						console.error('Can not compile "onLoad" function: ' + ex.message);
					}
				} else if(typeof this.config.onLoad == 'function') {
					fn = this.config.onLoad;
				}
				
				if(fn) {
					fn.call(this);
				}
			}
			
			this.$container.addClass('imgl-ready');
			this.ready = true;
			this.$container.trigger('imagelinks:ready', [this]);
		},
		
		_destroy: function() {
			this._unbind();
			this.controls.$image.remove();
			this.$container.removeClass('imgl-ready').removeClass('imgl-map').removeClass(this.config.className);
			this._setTheme(null);
			
			INSTANCE_COUNTER--;
			if(INSTANCE_COUNTER == 0) {
			}
		},
		
		_updateSize: function() {
			var ratio = this.imageHeight / this.imageWidth,
			imageWidth = this.controls.$image.width(),
			imageHeight = (this.config.height==null ? ratio * imageWidth : this.$container.height());
			
			if(this.config.height==null) {
				this.controls.$image.css({'height': imageHeight});
			}
			
			var ratioWidth = imageWidth/this.imageWidth,
			ratioHeight = imageHeight/this.imageHeight;
			
			for(var i=0;i<this.markers.length;i++) {
				var marker = this.markers[i];
				
				if(marker.cfg.responsive) {
					marker.$markerZoom.css({'transform':'scale(' + ratioWidth + ',' + ratioHeight + ')'});
				}
				
				if(marker.cfg.tooltip.responsive) {
					marker.$tooltipZoom.css({'transform':'scale(' + ratioWidth + ',' + ratioHeight + ')'});
				}
				
				if(marker.tooltipVisible) {
					this._showTooltip(marker);
				}
			}
		},
		
		_updatePlacement: function() {
			for(var i=0;i<this.markers.length;i++) {
				var marker = this.markers[i];
				
				if(marker.tooltipVisible) {
					this._showTooltip(marker);
				}
			}
		},
		
		_initMarkers: function() {
			if(this.config.markers) {
				for(var i=0; i<this.config.markers.length; i++) {
					var marker = this.config.markers[i];
					this._addMarker(marker);
				}
			}
		},
		
		_addMarker: function(marker) {
			marker.tooltip = $.extend(true, {}, this.defaultsTooltip, marker.tooltip);
			
			var marker = $.extend(true, {}, this.defaultsMarker, marker),
			$markerWrap = $('<li>').addClass('imgl-marker-wrap').addClass(marker.className),
			$markerPos = $('<div>').addClass('imgl-marker-pos').css({'top':marker.y + '%', 'left':marker.x + '%'}),
			$markerOffset = $('<div>').addClass('imgl-marker-offset'),
			$markerZoom = $('<div>').addClass('imgl-marker-zoom'),
			$marker = $('<div>').addClass('imgl-marker');
			
			if(marker.imgSrc) {
				$marker.css({'background-image':'url(' + marker.imgSrc + ')'});
			}
			
			if(marker.noevents) {
				$marker.addClass('imgl-noevents');
			}
			
			if(marker.tooltip.trigger == 'focus') {
				$marker.attr('tabindex',1);
			}
			
			if(!marker.autoWidth && marker.width!=null) {
				$marker.css({'width':marker.width});
			}
			
			if(!marker.autoHeight && marker.height!=null) {
				$marker.css({'height':marker.height});
			}
			
			if(marker.angle!=null) {
				$marker.css({'transform':'rotate(' + marker.angle + 'deg)'});
			}
			
			if(marker.data) {
				$marker.html(marker.data);
			} else if(marker.dataSelector) {
				var $el = $(marker.dataSelector);
				
				if($el.length) {
					$el.detach();
					$marker.append($el);
				}
			} else {
				//================================
				// special for wp version
				//================================
				var $el = this.$container.find('.imgl-store > .imgl-pin-' + (this.markers.length+1));
				if($el.length) {
					$el.detach();
					$marker.append($el);
				}
			}
			
			$markerWrap.append($markerPos.append($markerZoom.append($markerOffset.append($marker))));
			this.controls.$markers.append($markerWrap);
			
			this.controls.$lastMarker.detach();
			this.controls.$markers.append(this.controls.$lastMarker)
			
			//================================
			// TOOLTIP BEGIN
			marker.tooltip = $.extend(true, {}, this.defaultsTooltip, marker.tooltip);
			
			var $tooltipWrap = $('<div>').addClass('imgl-tooltip-wrap').addClass('imgl-hide').addClass(marker.tooltip.className),
			$tooltipPos = $('<div>').addClass('imgl-tooltip-pos').css({'top':marker.y + '%', 'left':marker.x + '%'}),
			$tooltipZoom = $('<div>').addClass('imgl-tooltip-zoom'),
			$tooltipOffset = $('<div>').addClass('imgl-tooltip-offset'),
			$tooltipForm = $('<div>').addClass('imgl-tooltip-form'),
			$tooltipArrow = $('<div>').addClass('imgl-tooltip-arrow'),
			$tooltipFrame = $('<div>').addClass('imgl-tooltip-frame'),
			$tooltip = $('<div>').addClass('imgl-tooltip');
			
			// set animation duration
			if((marker.tooltip.showAnimation || marker.tooltip.hideAnimation) && marker.tooltip.duration) {
				$tooltipForm.css({
					'animation-duration': marker.tooltip.duration + 'ms',
					'-webkit-animation-duration': marker.tooltip.duration + 'ms'
				});
			}
			
			if(marker.tooltip.data) {
				$tooltip.html(marker.tooltip.data);
			} else if(marker.tooltip.dataSelector) {
				var $el = $(marker.tooltip.dataSelector);
				if($el.length) {
					$el.detach();
					$tooltip.append($el);
				}
			} else {
				//================================
				// special for wp version
				//================================
				var $el = this.$container.find('.imgl-store .imgl-tt-' + (this.markers.length+1));
				if($el.length) {
					$el.detach();
					$tooltip.append($el);
				}
			}
			
			$tooltipWrap.append($tooltipPos.append($tooltipZoom.append($tooltipOffset.append($tooltipForm.append($tooltipFrame.append($tooltip), $tooltipArrow)))));
			this.controls.$tooltips.append($tooltipWrap);
			// TOOLTIP END
			//================================
			
			var marker = {
				$markerWrap: $markerWrap,
				$markerPos: $markerPos,
				$markerZoom: $markerZoom,
				$markerOffset: $markerOffset,
				$marker: $marker,
				$tooltipWrap: $tooltipWrap,
				$tooltipPos: $tooltipPos,
				$tooltipZoom: $tooltipZoom,
				$tooltipOffset: $tooltipOffset,
				$tooltipForm: $tooltipForm,
				$tooltipFrame: $tooltipFrame,
				$tooltip: $tooltip,
				tooltipVisible: false,
				cfg: marker
			};
			
			this._bindMarker(marker);
			this.markers.push(marker);
			
			this._updateBodyClick();
		},
		
		_bindMarker: function(marker) {
			if(marker.cfg.link) {
				marker.$marker.on('click', $.proxy(this._onMarkerClickLink, this, marker));
			}
			
			switch(marker.cfg.tooltip.trigger) {
				case 'hover': {
					marker.$marker.on('mouseenter', $.proxy(this._onMarkerEnterTooltip, this, marker));
					marker.$marker.on('mouseleave', $.proxy(this._onMarkerLeaveTooltip, this, marker));
					
					if(this.util.isMobile()) {
						marker.$marker.on('focus', $.proxy(this._onMarkerFocusTooltip, this, marker)); //???
						marker.$marker.on('blur', $.proxy(this._onMarkerBlurTooltip, this, marker));
					}
				} break;
				case 'focus': {
					marker.$marker.on('focus', $.proxy(this._onMarkerFocusTooltip, this, marker));
					marker.$marker.on('blur', $.proxy(this._onMarkerBlurTooltip, this, marker));
				} break;
				case 'click':
				case 'clickbody': {
					marker.$marker.on('click', $.proxy(this._onMarkerClickTooltip, this, marker));
				} break;
			}
		},
		
		_deleteMarker: function(markerIndex) {
			if(markerIndex>=0 && markerIndex<this.markers.length) {
				var marker = this.markers[markerIndex];
				
				marker.$markerWrap.remove();
				marker.$tooltipWrap.remove();
				
				this.markers.splice(markerIndex, 1);
				
				return true;
			}
			return false;
		},
		
		_updateBodyClick: function() {
			this.bodyClick = false;
			for(var i=0;i<this.markers.length;i++) {
				var marker = this.markers[i];
				if(marker.cfg.tooltip.trigger == 'clickbody' || marker.cfg.tooltip.trigger == 'focus') {
					this.bodyClick = true;
					break;
				}
			}
		},
		
		_getViewportOffset: function($el) {
			var $window = $(window),
			scrollLeft = $window.scrollLeft(),
			scrollTop = $window.scrollTop(),
			rect = $el.get(0).getBoundingClientRect(),
			offset = $el.offset();
			
			return {
				left: offset.left - scrollLeft,
				top: offset.top - scrollTop,
				bottom: Math.max(document.documentElement.clientHeight, window.innerHeight || 0) - offset.top + scrollTop - rect.height,
				right: Math.max(document.documentElement.clientWidth, window.innerWidth || 0) - offset.left + scrollLeft - rect.width
			};
		},
		
		_showTooltip: function(marker, animate, placement, force) {
			if(!marker.cfg.tooltip.active) {
				return;
			}
			
			var ratio = this.imageHeight / this.imageWidth,
			imageWidth = this.$container.width(),
			imageHeight = (this.config.height==null ? ratio * imageWidth : this.$container.height()),
			ratioWidth = imageWidth/this.imageWidth,
			ratioHeight = imageHeight/this.imageHeight,
			offsetTop  = (marker.cfg.tooltip.trigger == 'hover' && marker.cfg.tooltip.followCursor ? 5 : (!marker.cfg.autoHeight && marker.cfg.height ? marker.cfg.height : marker.$marker.height()) * (marker.cfg.responsive && !marker.cfg.tooltip.responsive ? ratioHeight : 1)/2),
			offsetLeft = (marker.cfg.tooltip.trigger == 'hover' && marker.cfg.tooltip.followCursor ? 5 : (!marker.cfg.autoWidth && marker.cfg.width  ? marker.cfg.width  : marker.$marker.width())  * (marker.cfg.responsive && !marker.cfg.tooltip.responsive ? ratioWidth  : 1)/2),
			marginTop = marker.cfg.tooltip.offset.y,
			marginLeft = marker.cfg.tooltip.offset.x;
			
			
			animate = animate || false;
			placement = placement || marker.cfg.tooltip.placement;
			force = force || false;
			
			marker.$tooltipWrap.removeClass(marker.$tooltipWrap.data('placement'));
			marker.$tooltipWrap.addClass('imgl-tooltip-' + placement).data('placement', 'imgl-tooltip-' + placement);
			
			switch(placement) {
				case 'top-left': {
					offsetTop  = -offsetTop;
					offsetLeft = -offsetLeft;
					marginTop  = -marginTop;
					marginLeft =  marginLeft;
				} break;
				case 'top': {
					offsetTop  = -offsetTop;
					offsetLeft = 0;
					marginTop  = -marginTop;
					marginLeft =  0;
				} break;
				case 'top-right': {
					offsetTop  = -offsetTop;
					offsetLeft =  offsetLeft;
					marginTop  = -marginTop;
					marginLeft = -marginLeft;
				} break;
				case 'right-top': {
					offsetTop  = -offsetTop;
					offsetLeft =  offsetLeft;
					marginTop  =  marginTop;
					marginLeft =  marginLeft;
				} break;
				case 'right': {
					offsetTop  = 0;
					offsetLeft = offsetLeft;
					marginTop  = 0;
					marginLeft = marginLeft;
				} break;
				case 'right-bottom': {
					offsetTop  =  offsetTop;
					offsetLeft =  offsetLeft;
					marginTop  = -marginTop;
					marginLeft =  marginLeft;
				} break;
				case 'bottom-right': {
					offsetTop  =  offsetTop;
					offsetLeft =  offsetLeft;
					marginTop  =  marginTop;
					marginLeft = -marginLeft;
				} break;
				case 'bottom': {
					offsetTop  = offsetTop;
					offsetLeft = 0;
					marginTop  = marginTop;
					marginLeft = 0;
				} break;
				case 'bottom-left': {
					offsetTop  =  offsetTop;
					offsetLeft = -offsetLeft;
					marginTop  =  marginTop;
					marginLeft =  marginLeft;
				} break;
				case 'left-bottom': {
					offsetTop  =  offsetTop;
					offsetLeft = -offsetLeft;
					marginTop  = -marginTop;
					marginLeft = -marginLeft;
				} break;
				case 'left': {
					offsetTop  =  0;
					offsetLeft = -offsetLeft;
					marginTop  =  0;
					marginLeft = -marginLeft;
				} break;
				case 'left-top': {
					offsetTop  = -offsetTop;
					offsetLeft = -offsetLeft;
					marginTop  =  marginTop;
					marginLeft = -marginLeft;
				} break;
				default: {
					offsetTop = offsetLeft = (marker.cfg.tooltip.followCursor ? 5 : 0);
				}
				break;
			}
			marker.$tooltipOffset.css({'margin-top':offsetTop, 'margin-left':offsetLeft});
			
			if(marker.cfg.tooltip.followCursor && marker.cfg.tooltip.trigger == 'hover') {
				marker.$marker.on('mousemove', $.proxy(this._onMarkerMouseMoveTooltip, this, marker));
				//marker.$tooltip.on('mousemove', $.proxy(this._onTooltipMouseMove, this, marker));
				marker.$tooltipWrap.addClass('imgl-noevents-hard');
			}
			
			marker.$tooltipPos.css({'top':marker.cfg.y + '%','left':marker.cfg.x + '%','margin-top':marginTop,'margin-left':marginLeft});
			
			if(!marker.tooltipVisible) {
				marker.tooltipVisible = true;
				marker.$tooltipWrap.addClass('imgl-show').removeClass('imgl-hide');
				
				var zindex = (this.tooltipZindex.length > 0 ? Math.max.apply(null, this.tooltipZindex) + 1 : 1);
				this.tooltipZindex.push(zindex);
				marker.$tooltipPos.css('z-index', zindex);
				
				if(animate && marker.cfg.tooltip.showAnimation) {
					marker.$tooltipForm.removeClass(marker.cfg.tooltip.showAnimation).removeClass(marker.cfg.tooltip.hideAnimation);
					marker.$tooltipForm.addClass(marker.cfg.tooltip.showAnimation);
					
					marker.$tooltipForm.one(this.util.animationEvent(), $.proxy(function(marker, e) {
						var $tooltipForm = $(e.target);
						$tooltipForm.removeClass(marker.cfg.tooltip.showAnimation);
					}, this, marker));
				}
			}
			
			if(!force && marker.cfg.tooltip.smart) {
				var markerOffset = this._getViewportOffset(marker.$markerOffset),
				tooltipOffset = this._getViewportOffset(marker.$tooltipOffset),
				markerRect = marker.$markerOffset.get(0).getBoundingClientRect(),
				tooltipRect = marker.$tooltipOffset.get(0).getBoundingClientRect();
				
				switch(placement) {
					case 'top-left': {
						if(tooltipOffset.right<0) {
							if(markerOffset.left + markerRect.width - tooltipRect.width > 0) {
								placement = placement.replace('-left', '-right');
							} else {
								placement = placement.replace('-left', '');
							}
						}
						if(tooltipOffset.top<0) {
							placement = placement.replace('top', 'bottom');
						}
					} break;
					case 'top': {
						if(tooltipOffset.right<0) {
							if(markerOffset.left + markerRect.width - tooltipRect.width > 0) {
								placement = placement + '-right';
							}
						}
						if(tooltipOffset.left<0) {
							if(markerOffset.right + markerRect.width - tooltipRect.width > 0) {
								placement = placement + '-left';
							}
						}
						if(tooltipOffset.top<0) {
							placement = placement.replace('top', 'bottom');
						}
					} break;
					case 'top-right': {
						if(tooltipOffset.left<0) {
							if(markerOffset.right + markerRect.width - tooltipRect.width > 0) {
								placement = placement.replace('-right', '-left');
							} else {
								placement = placement.replace('-right', '');
							}
						}
						if(tooltipOffset.top<0) {
							placement = placement.replace('top', 'bottom');
						}
					} break;
					case 'right-top': {
						if(tooltipOffset.top<0) {
							if(markerRect.height > tooltipRect.height) {
								placement = placement.replace('-top', '-bottom');
							}
						}
						if(tooltipOffset.right<0) {
							placement = placement.replace('right', 'left');
						}
					} break;
					case 'right': {
						if(tooltipOffset.top<0) {
							if(markerRect.height > tooltipRect.height) {
								placement = placement + '-bottom';
							} else {
								placement = placement + '-top';
							}
						}
						if(tooltipOffset.bottom<0) {
							if(markerRect.height > tooltipRect.height) {
								placement = placement + '-top';
							} else {
								placement = placement + '-bottom';
							}
						}
						if(tooltipOffset.right<0) {
							placement = placement.replace('right', 'left');
						}
					} break;
					case 'right-bottom': {
						if(tooltipOffset.bottom<0) {
							if(markerRect.height > tooltipRect.height) {
								placement = placement.replace('-bottom', '-top');
							}
						}
						if(tooltipOffset.right<0) {
							placement = placement.replace('right', 'left');
						}
					} break;
					case 'bottom-right': {
						if(tooltipOffset.left<0) {
							if(markerOffset.right + markerRect.width - tooltipRect.width > 0) {
								placement = placement.replace('-right', '-left');
							} else {
								placement = placement.replace('-right', '');
							}
						}
						if(tooltipOffset.bottom<0) {
							placement = placement.replace('bottom', 'top');
						}
					} break;
					case 'bottom': {
						if(tooltipOffset.right<0) {
							if(markerOffset.left + markerRect.width - tooltipRect.width > 0) {
								placement = placement + '-right';
							}
						}
						if(tooltipOffset.left<0) {
							if(markerOffset.right + markerRect.width - tooltipRect.width > 0) {
								placement = placement + '-left';
							}
						}
						if(tooltipOffset.top<0) {
							placement = placement.replace('bottom', 'top');
						}
					} break;
					case 'bottom-left': {
						if(tooltipOffset.right<0) {
							if(markerOffset.left + markerRect.width - tooltipRect.width > 0) {
								placement = placement.replace('-left', '-right');
							} else {
								placement = placement.replace('-left', '');
							}
						}
						if(tooltipOffset.bottom<0) {
							placement = placement.replace('bottom', 'top');
						}
					} break;
					case 'left-bottom': {
						if(tooltipOffset.bottom<0) {
							if(markerRect.height > tooltipRect.height) {
								placement = placement.replace('-bottom', '-top');
							}
						}
						if(tooltipOffset.left<0) {
							placement = placement.replace('left', 'right');
						}
					} break;
					case 'left': {
						if(tooltipOffset.top<0) {
							if(markerRect.height > tooltipRect.height) {
								placement = placement + '-bottom';
							} else {
								placement = placement + '-top';
							}
						}
						if(tooltipOffset.bottom<0) {
							if(markerRect.height > tooltipRect.height) {
								placement = placement + '-top';
							} else {
								placement = placement + '-bottom';
							}
						}
						if(tooltipOffset.left<0) {
							placement = placement.replace('left', 'right');
						}
					} break;
					case 'left-top': {
						if(tooltipOffset.top<0) {
							if(markerRect.height > tooltipRect.height) {
								placement = placement.replace('-top', '-bottom');
							}
						}
						if(tooltipOffset.left<0) {
							placement = placement.replace('left', 'right');
						}
					} break;
				}
				this._showTooltip(marker, false, placement, true, false);
			}
		},
		
		_hideTooltip: function(marker) {
			marker.$marker.off('mousemove', $.proxy(this._onMarkerMouseMoveTooltip, marker, this));
			marker.$tooltipWrap.removeClass('imgl-noevents-hard');
			
			if(marker.tooltipVisible) {
				marker.tooltipVisible = false;
				
				if(marker.cfg.tooltip.hideAnimation) {
					marker.$tooltipForm.removeClass(marker.cfg.tooltip.showAnimation).removeClass(marker.cfg.tooltip.hideAnimation);
					marker.$tooltipForm.addClass(marker.cfg.tooltip.hideAnimation);
					
					marker.$tooltipForm.one(this.util.animationEvent(), $.proxy(function(marker, e) {
						var $tooltipForm = $(e.target);
						$tooltipForm.removeClass(marker.cfg.tooltip.hideAnimation);
						if(!marker.tooltipVisible) {
							this.__hideTooltip(marker);
						}
					}, this, marker));
				} else {
					this.__hideTooltip(marker);
				}
			}
		},
		
		__hideTooltip: function(marker) {
			marker.$tooltipWrap.addClass('imgl-hide').removeClass('imgl-show');
			
			var index = this.tooltipZindex.indexOf(parseInt(marker.$tooltipPos.css('z-index'),10));
			if(index > -1) {this.tooltipZindex.splice(index, 1);}
			marker.$tooltipPos.css('z-index','');
			
			// if we want to disable media content fully //???
			marker.$tooltip.detach();
			marker.$tooltip.get(0).offsetHeight;
			marker.$tooltip.appendTo(marker.$tooltipFrame);
		},
		
		_onMarkerClickLink: function(marker, e) {
			if(marker.cfg.link) {
				window.open(marker.cfg.link, (marker.cfg.linkNewWindow ? '_blank' : '_self'));
			}
		},
		
		_onMarkerEnterTooltip: function(marker, e) {
			this._showTooltip(marker, true);
		},
		
		_onMarkerLeaveTooltip: function(marker, e) {
			var from = e.relatedTarget || e.toElement;
			
			if((!marker.cfg.tooltip.interactive || marker.$tooltipWrap.has(from).length === 0 && !marker.$tooltipWrap.is(from)) && !marker.$marker.is(from) ) {
				this._hideTooltip(marker);
			} else {
				marker.$tooltipWrap.one('mouseleave', $.proxy(this._onMarkerLeaveTooltip, this, marker));
			}
		},
		
		_onMarkerMouseMoveTooltip: function(marker, e) {
			var rc = this.controls.$stage.get(0).getBoundingClientRect(),
			y = ((e.clientY - rc.y) / rc.height)*100,
			x = ((e.clientX - rc.x) / rc.width)*100;
			
			marker.$tooltipPos.css({'top':y + '%', 'left':x + '%'});
		},
		
		_onMarkerFocusTooltip: function(marker, e) {
			this._showTooltip(marker, true);
		},
		
		_onMarkerBlurTooltip: function(marker, e) {
			this._hideTooltip(marker);
		},
		
		_onMarkerClickTooltip: function(marker, e) {
			if(marker.tooltipVisible) {
				this._hideTooltip(marker);
			} else {
				this._showTooltip(marker, true);
			}
		},
		
		_onBodyClickTooltip: function(e) {
			if(!this.bodyClick) {
				return;
			}
			
			var from = e.target || e.relatedTarget || e.toElement;
			
			for(var i=0;i<this.markers.length;i++) {
				var marker = this.markers[i];
				
				if(marker.$markerWrap.has(from).length > 0 || marker.$markerWrap.is(from)) {
					continue;
				}
				
				if(marker.tooltipVisible && (marker.cfg.tooltip.trigger == 'clickbody' || (marker.cfg.tooltip.trigger == 'focus' && !marker.$marker.is(':focus')))) {
					if(marker.tooltipVisible && (!marker.cfg.tooltip.interactive || marker.$tooltipWrap.has(from).length === 0 && !marker.$tooltipWrap.is(from))) {
						this._hideTooltip(marker);
					}
				}
			}
		},
		
		_onLastMarkerFocus: function(e) {
			if(this.markers.length) {
				this.markers[0].$marker.focus();
			}
		},
		
		_onResize: function() {
			if(this.ready) {
				var _self = this;
				
				clearTimeout(this.timerIdResize);
				this.timerIdResize = setTimeout(function() {_self._updateSize();}, this.config.delayResize);
			}
		},
		
		_onScroll: function() {
			if(this.ready) {
				var _self = this;
				
				clearTimeout(this.timerIdScroll);
				this.timerIdScroll = setTimeout(function() {_self._updatePlacement();}, 100);
			}
		},
		//=============================================
		// Public Methods
		//=============================================
		setTheme: function(theme) {
			this._setTheme(theme);
		},
		
		addMarker: function(marker) {
			this._addMarker(marker);
		},
		
		deleteMarker: function(markerIndex) {
			return this._deleteMarker(markerIndex);
		},
	}
	//=============================================
	// Init jQuery Plugin
	//=============================================
	/**
	 * @param CfgOrCmd - config object or command name
	 * @param CmdArgs - some commands may require an argument
	 * List of methods:
	 * $("#imagelinks").imagelinks("destroy")
	 * $("#imagelinks").imagelinks("instance")
	 */
	$.fn.imagelinks = function(CfgOrCmd, CmdArgs) {
		if (CfgOrCmd == 'instance') {
			var $container = $(this),
			instance = $container.data(ITEM_DATA_INSTANCE);
			
			if (!instance) {
				console.error('Calling "instance" method on not initialized instance is forbidden');
				return;
			}
			
			return instance;
		}
		
		return this.each(function() {
			var $container = $(this),
			instance = $container.data(ITEM_DATA_INSTANCE),
			jsonSrc = $container.data('json-src'),
			imgSrc = $container.data('img-src'),
			options = $.isPlainObject(CfgOrCmd) ? CfgOrCmd : (jsonSrc ? null : {});
			
			if(CfgOrCmd == 'destroy') {
				if (!instance) {
					console.error('Calling "destroy" method on not initialized instance is forbidden');
					return;
				}
				
				$container.removeData(ITEM_DATA_INSTANCE);
				instance._destroy();
				
				return;
			}
			
			if(instance) {
				$container.removeData(ITEM_DATA_INSTANCE);
				instance._destroy();
				instance = null;
			}
			
			function init() {
				var config = $.extend(true, {}, ImageLinks.prototype.defaults, options);
				
				//=============================================
				// if the imgSrc parameter is not set, we try to use value from data-img-src attribute
				if(config.imgSrc == null && imgSrc) {
					config.imgSrc = imgSrc;
				}
				//=============================================
				
				if(config.onPreInit) {
					var fn = null;
					if(typeof config.onPreInit == 'string') {
						try {
							fn = new Function('config', config.onPreInit);
						} catch(ex) {
							console.error('Can not compile "onPreInit" function: ' + ex.message);
						}
					} else if(typeof config.onPreInit == 'function') {
						fn = config.onPreInit;
					}
					
					if(fn) {
						fn.call(this, config);
					}
				}
				
				instance = new ImageLinks($container, config);
				$container.data(ITEM_DATA_INSTANCE, instance);
			}
			
			// options have more priority than json
			if(options == null) {
				$.ajax({
					url: jsonSrc,
					type: 'GET',
					dataType: 'json'
				}).done(function(data) {
					options = $.isPlainObject(data) ? data : {};
				}).fail(function() {
					options = {};
				}).always(function() {
					init();
				});
			} else {
				init();
			}
		});
	}
	
	$('.imgl-map').imagelinks();
})(jQuery, window, document);