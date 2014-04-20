UI Helper Classes
=================

jQuery UI is nice, but I prefer some abstracting code to define my actual UI 
components. The module exports can function as a table of contents:

[Main file](index.coffee.md)

Tabs
----

#### Class Tabs

use this class to generate a tabs environment on an arbitrary element 
in your DOM. Simply identify the element via a jQuery selection.

    class Tabs
      tabs = {}
      constructor: (@id) ->
        $("#{@id}").append(
          "<ul></ul>
          <div class=\"ui-layout-content ui-widget-content\"></div></div>"
        )
        $("#{@id}").tabs()
          .find(".ui-tabs-nav").sortable({ axis: 'x', zIndex: 2 })

      addTab: (tabid, title) ->
        if tabs[tabid]!=undefined
          tabs[tabid].select()
          return tabs[tabid]
        if (@size()==0) then @hideWelcome()
        newtab = new Tab(tabid,title, this)
        tabs[tabid] = newtab
        newtab.select()
        newtab

      refresh: ->
        console.log("Refreshing: #{@id}")
        $("#{@id}").tabs("refresh")

      size: ->
        (k for own k of tabs).length
      unlist: (tabid) ->
        delete tabs[tabid]
        if (@size()==0) then @showWelcome()

      getTabIDs: ->
        result = []
        for own key, value of tabs
          result.push(key)
        result

      setWelcome: (@welcome) =>
        if (@size()==0)
          @hideWelcome()
          @showWelcome()

      showWelcome: =>
        if @welcome==undefined then return
        $(".ui-layout-content").append(@welcome)

      hideWelcome: =>
        if @welcome==undefined then return
        $(".ui-layout-content").empty()


    class Tab
      constructor: (@tabid, @title, @tabs) ->
        uuid = require('uuid')
        @domid = uuid.v4()
        $("#{@tabs.id} > ul").append("<li class='li-#{@domid}'>
          <a href=\"##{@domid}\">#{@title}</a>
          <span class='ui-icon ui-icon-circle-close
          ui-closable-tab close-#{@domid}'></li>")
        $("#{@tabs.id} .ui-layout-content").append("<div id =\"#{@domid}\"></div>")
        @tabs.refresh()
        $(".close-#{@domid}").on "click", =>
          @remove()


      setContent: (content) ->
        $("##{@domid}").html(content)
        @tabs.refresh()

      select: ->
        index=$("#{@tabs.id} ul li").index($("li[aria-controls='#{@domid}']"))
        $( "#{@tabs.id}" ).tabs("option", "active", index)


      remove: ->
        console.log("Close: " + @tabid)
        $(".li-#{@domid}").remove()
        $("#" + @domid).remove()
        @tabs.refresh()
        @tabs.unlist(@tabid)
        return


    module.exports.Tabs = Tabs
