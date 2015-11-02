---
layout:     post
title:      Hintholder : 모든 브라우저를 위한 placeholder
date:       2015-11-02
summary:    크롬, 파폭, 사파리 등 모던 브라우저부터 구버전 IE 까지 사용할 수 있는 placeholder
categories: dev
---
구버전 IE에서 placeholder를 반드시 사용해야 하는 경우가 있어서 만들었던 jQuery용 placeholder 라이브러리.

[https://github.com/lainfox/Hintholder](https://github.com/lainfox/Hintholder)

placeholder가 있는 input에 적용되며 data 속성 어트리뷰트에 따라 몇몇 옵션을 사용할 수 있다.

`data-compatible='true'` : 로 하면 모든 브라우저에서 사용. 디폴트는 false 값으로 IE에서만
`data-opacity='true'` : input에 포커스가 들어가 있을때 흐리게 보이기. false는 placeholder 텍스트가 안보임.
`data-color='DodgerBlue'` : css에서 사용하는 컬러값. ex) DodgerBlue, #1E90FF, rgb(30,144,255)


예재 데모 페이지는 조만간 작성해서 공개하겠어요 ;p

---- 

아래는 풀 소스코드

    /* ============================================================
     * Hintholder 0.9.1
     * Placeholder for legacy IE or all mordern browser
     * ============================================================
     * Copyright 2015 lainfox
     * Licensed under MIT
     * Released on: July 02, 2015
     * ============================================================ */
    !function($){
      'use strict';
    
      var Hintholder = function(element, options) {
        this.options = $.extend({}, Hintholder.DEFAULTS, options)
        this.$element = $(element)
        this.label_text  = this.$element.attr('placeholder')
        this.label_for   = this.$element.attr('id')
    
        this.$element
          .on('focusin.hint.placeholder', $.proxy(this._inFocus, this))
          .on('focusout.hint.placeholder', $.proxy(this._outFocus, this))
          .on('keydown.hint.placeholder', $.proxy(this._keyPress, this))
          .on('paste.hint.placeholder', $.proxy(this._keyPress, this))
    
        this.setLabel()
      }
    
      Hintholder.DEFAULTS = {
        compatible: false // false = only IE, true = all mordern browser
        ,opacity: true // true = lower opacity at focus, false = hide at focus
        ,color: null // placeholder color hex value #d1d1d1
        ,icon: null // icon class name in CSS declaration :: not yet implemented
      }
      Hintholder.RESET = 'hint-focus hint-hide'
    
      Hintholder.prototype._inFocus = function() {
        if( this.options.opacity ) {
          this.$label.addClass('hint-focus')
        }
        else {
          this.$label.addClass('hint-hide')
        }
      }
      Hintholder.prototype._outFocus = function() {
        if( !this.$element.val() )
          this.$label.removeClass( Hintholder.RESET)
      }
      Hintholder.prototype._keyPress = function() {
        this.$label.addClass('hint-hide')
      }
      Hintholder.prototype.setLabel = function() {
        var $el = this.$element
    
        this.$label =
          $('<label />')
            .attr('for', this.label_for)
            .addClass('hint-holder')
            .text( this.label_text )
            .css({
              'font-size': $el.css('font-size'),
              'line-height': $el.css('height'),
              'margin-left': $el.css('margin-left'),
              'margin-top': $el.css('margin-top'),
              'padding-left': $el.css('padding-left'),
              'height': $el.css('height')
            })
            .on('click.hint.placeholder', function(){ $el.focus() } )
    
        if( this.options.color ) {
          this.$label.css('color', this.options.color);
        }
    
        // Attach label
        if( !_placeholderIsSupported() ) { // only IE
          $el.before( this.$label )
        }
        else if( this.options.compatible === true ) { // All
          $el.addClass('hint-holder-input')
          $el.before( this.$label )
        }
    
        // hide hintholder when init value is not empty
        if( $el.val() ) {
          this.$label.addClass('hint-hide')
        }
      }
    
      function _placeholderIsSupported() {
        var test = document.createElement('input');
        return ('placeholder' in test);
      }
    
      function Init(option) {
        return this.each(function(){
          var $this       = $(this)
          ,   data        = $(this).data('hint.placeholder')
          ,   options     = typeof option == 'object' && option
    
          // console.log( data )
          if( !data ) $this.data('hint.placeholder', (data = new Hintholder(this, options) ))
        })
      }
    
      var old = $.fn.hintholder
      $.fn.hintholder             = Init
      $.fn.hintholder.Constructor = Hintholder
      $.fn.hintholder.noConflict = function() {
        $.fn.hintholder = old
        return this
      }
    
      $(window).on('load', function() {
        $('input[placeholder]').each(function() {
          var $hintholder = $(this)
          var data = $hintholder.data()
    
          Init.call( $hintholder, data )
        })
      })
    
    }(jQuery)
