Hands Down
==========

    ui = require ("../js/ui")
    gui = require('nw.gui')
    fs = require("fs")
    path = require("path")
    marked = require('marked')
      
    # https://github.com/rogerwang/node-webkit/wiki/Transfer-objects-between-window-and-node
    global.$ = $
    

    



    $ ->

The tab environment is created by a helper class, 
defined in [ui.coffee.md](ui.coffee.md)

      tabs = new ui.Tabs(".ui-layout-center")
      tabs.setWelcome("
        <div class='.welcome' style='
            text-align:center;
            font-size:10pt;
            color:#CC7A29;
            margin:40px;
            padding:80px;
            border: 2px dashed gray;
           -webkit-border-radius: 40px;
          '>
          <h2>HandsDown</h2>
          <h1><i class='fa fa-hand-o-down fa-3x'></i></h1>
          <p style='margin-left:auto;margin-right:auto;width:400px;text-align:left'>
            Hands down, this is the simplest <b>viewer for Markdown</b>
            (and other) files you have ever seen.
          </p>
          <h2 style='
            margin:40px;
            padding:20px;
            border: 0px dashed gray;
            -webkit-border-radius: 20px;
            background-color:#CC7A29;
            color:#FFFFFF;
         '>Drag and drop your files here...</h2>
          <p style='margin-left:auto;margin-right:auto;width:400px;text-align:left'>
            ... and they will automatically
            update whenever you change them in your editor.
          </p>
          <p style='margin-left:auto;margin-right:auto;width:400px;text-align:left'>
            <small><i>
            &copy; 2014 Kai Eckert,
            <a href='https://github.com/kaiec/HandsDown' style='color:#CC7A29'>
              github.com/kaiec/HandsDown
            </a>
            </i></small>
          </p>
        </div>")
      layout = $('body').layout()
      marked.setOptions({
        highlight: (code) ->
          hljs = require('highlight.js')
          hljs.configure({classPrefix: ''})
          return hljs.highlightAuto(code).value
      })

      interceptLinks = (tab) ->
        $("##{tab.domid} a").click((e)->
          e.preventDefault()
          href = $(this).attr('href')
          console.log("Click: " + href)
          file = path.dirname(tab.tabid) + path.sep + href
          if fs.existsSync(file)
            openFile(file)
          else
            gui.Shell.openExternal(href)
        )
      

      fillTab = (f, tab) ->
        fs.readFile(f, "utf8", (err, data) ->
          if (err) then throw err
          if (path.extname(f)==".md")
            marked(data, (err,data)->
              if (err) then throw err
              tab.setContent(data)
              interceptLinks(tab)
              console.log("Added: " + f)
            )
          else
            escape = require('escape-html')
            hljs = require('highlight.js')
            hljs.configure({classPrefix: ''})
            data = hljs.highlightAuto(data).value
            tab.setContent("<pre>" + data + "</pre>")
            console.log("Added: " + f)
        )

      syncTab = (f, tab) ->
        return (event, file) ->
          console.log(file + " " + event + " " + f + " " + tab.tabid)
          if event=="change"
            fillTab(f, tab)
            tab.select()



      openFile = (f) ->
        f = path.normalize(f)
        tab = tabs.addTab(f, path.basename(f))
        fillTab(f, tab)
        tab.select()
        fs.watch(f, { persistent: true }, syncTab(f,tab))



      files = []
      if (localStorage.files)
        files = JSON.parse(localStorage.files)

      for f in files
        f = fs.realpathSync(f)
        openFile(f)
      
     

      window.ondragover = (e) ->
        e.preventDefault()
        false

      window.ondrop = (e) ->
        e.preventDefault()
        false

      holder = window
      holder.ondragover = ->
        @className = "hover"
        false

      holder.ondragend = ->
        @className = ""
        false

      holder.ondrop = (e) ->
        e.preventDefault()
        i = 0
        while i < e.dataTransfer.files.length
          f =  e.dataTransfer.files[i].path
          openFile(f)
          ++i
        false

      win = gui.Window.get()
      win.on('close', (e) ->
        localStorage.files = JSON.stringify(tabs.getTabIDs())
        this.close(true)
      )
