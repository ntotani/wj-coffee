fs    = require 'fs'
path  = require 'path'
exec  = require('child_process').exec

option '-o', '--output [DIR]', 'directory for compiled code'
option '-c', '--compile [DIR]', 'directory for sorce code'

task 'watch', 'watches and compiles coffee file', (options) ->
  target = options.compile or 'app/libs/client'
  output = options.output or 'app/webroot/js/app.js'
  watchs = new Array()

  init = () ->
    return

  run = (cb) ->
    console.log """\n
--------------- \x1b[33mStart to watch directory and compiling\x1b[39m -------------------
  target directory -> #{target}
  output directory -> #{output}
--------------------------------------------------------------------------
\x1b[36mWatch File List:\x1b[39m
    """
    watch_for_change(target, cb)

  watch_for_change = (target, cb) ->
    fs.stat target, (err, stats) ->
      if stats.isDirectory()
        watch_for_change_in_dir(target, cb)
      else if stats.isFile()
        watch(target)
      else
        return

  watch_for_change_in_dir = (dir, cb) ->
    fs.readdir dir, (err, files) ->
      files.forEach (file) ->
        current = "#{dir}/#{file}"
        watch_for_change(current, cb)
    return

  watch = (file, cb) ->
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
    return
  
  # ファイル名順に結合
  compile= () ->
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

  run()
