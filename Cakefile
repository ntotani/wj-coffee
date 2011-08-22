fs    = require 'fs'
path  = require 'path'
exec  = require('child_process').exec

option '-o', '--output [DIR]', 'directory for compiled code'
option '-c', '--compile [DIR]', 'directory for sorce code'

task 'watch', 'watches and compiles coffee file', (options) ->
  target = options.compile or 'app/libs/client'
  output = options.output or 'app/webroot/js/app.js'
  watchs = new Array()

  main = () -> 
    fs.stat target, (err, stats) ->
      if stats.isDirectory()
        searchDirectory(target)
      else if stats.isFile()
        watchFile(target)
      else
        return

  searchDirectory = (dir) ->
    fs.readdir dir, (err, files) ->
      files.forEach (file) ->
        current = "#{dir}/#{file}"
        fs.stat current, (err, stats) ->
          if stats.isDirectory()
            searchDirectory(current)
          else if stats.isFile()
            watchFile(current)

  watchFile = (file) ->
    if file.match(/.coffee$/) == null
      return

    console.log "#{file}"

    watchs.push(file)

    fs.watchFile file, (cur, prev) ->
      if file && +cur.mtime != +prev.mtime
        text  = "\n\x1b[36mCompiling...\x1b[39m\n"
        text += "Date:          #{cur.mtime}\n"
        text += "Modified File: #{file}\n"
        console.log text
        compile()

  # ファイル名順に結合する為、監視ファイルは頭に(数字.)付与推奨
  compile = () ->
    filelist = []
    watchs.forEach (watch) ->
      filelist.push({'file' : path.basename(watch), 'path': watch})
    # key でソート
    key = 'file'
    filelist.sort (b1, b2) ->
      return b1[key] > b2[key] ? 1 : -1
    
    pathlist = []
    filelist.forEach (file) ->
      pathlist.push(file.path)

    files = pathlist.join(" ")
    exec "coffee -bj #{output} -c #{files}" ,(err, stdout, stderr) ->
      # output error & exit
      if !err?
        console.log "\x1b[35mCompleted!"
        fs.stat output, (err, stats) ->
          if(+stats.atime == +stats.mtime)
            console.log "Create: #{output}\x1b[39m"
          else
            console.log "Overwrite: #{output}\x1b[39m"
      else
        console.log "\x1b[31mFailed!\n" + stdout + stderr + "\x1b[39m"

    
  # initial compile
  console.log """\n
--------------- \x1b[33mStart to watch directory and compiling\x1b[39m -------------------
  target directory -> #{target}
  output directory -> #{output}
--------------------------------------------------------------------------
\x1b[36mWatch File List:\x1b[39m
  """
  # watch
  main()

 
