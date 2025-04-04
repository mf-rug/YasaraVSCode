# YasaraVSCode
Execute Yasara commands from VSCode


## installation
```
# create the file to which current selection is written
touch "$HOME/.vscode/save_hl.py"

# shell script that writes the file
echo \
'#!/bin/bash
pbpaste > ~/.vscode/save_hl.py' \
> "$HOME/.vscode/append_from_clipboard.sh"

# make shell script executable
chmod +x append_from_clipboard.sh

# install multi-command.
# if `code` isn't available, install manually within VSCode
code --install-extension ryuta46.multi-command

# add a keyboard shortcut to VSCode, adjust / perform manually if desired
echo \
'
[
{
  "key": "cmd+alt+a",
  "command": "extension.multiCommand.execute",
  "args": {
    "sequence": [
      "editor.action.clipboardCopyAction",
      {
        "command": "workbench.action.terminal.sendSequence",
        "args": {
          "text": "$HOME/.vscode/append_from_clipboard.sh\u000D"
        }
      }
    ]
  },
  "when": "editorTextFocus"
}
]
' >> "$HOME/Library/Application Support/Code/User/keybindings.json"

#Then create a yasara plugin
cat <<EOF > /Applications/YASARA.app/Contents/yasara/plg/
# YASARA PLUGIN
# TOPIC:       Python
# TITLE:       PythonVSCode
# AUTHOR:      M.J.L.J. Fürst
# LICENSE:     GPL (www.gnu.org)
# DESCRIPTION: This plugin allows executing python code in a YASARA session from VSCode
#
 
"""
MainMenu: Options
  PullDownMenu: PythonVSCode
    Request: Python
"""

from yasara import *
import os
import time

# get paths
home = os.environ["HOME"]
file_to_watch = os.path.join(home, ".vscode/save_hl.py")
closeplg = os.path.join(home, '.vscode/closeplg')

# delete existing helper files
if (os.path.exists(file_to_watch)):
    os.remove(file_to_watch)
if (os.path.exists(closeplg)):
    os.remove(file_to_watch)

with open(closeplg, 'a'):
    os.utime(closeplg, None)

with open(file_to_watch, 'a'):
    os.utime(file_to_watch, None)

# Make Button to quit plugin
Console('OFF')
img = MakeImage("Buttons",topcol="None",bottomcol="None")
ShowImage(img,alpha=85,priority=1)
PrintImage(img) 
Font("Arial",height=7,color="black")
ShowButton("PyVSC",x='80%', y='0.5%',color="White", height=29, action=f'DelFile {closeplg}')

# Show info
ShowMessage('PythonVSCode plugin launched. To quit, press button above HUD.')
Wait(500)
HideMessage()
PrintCon()
Console('ON')

# run main loop
last_mtime = 0
while os.path.exists(closeplg):
    try:
        mtime = os.path.getmtime(file_to_watch)
        if mtime != last_mtime:
            last_mtime = mtime
            with open(file_to_watch, "r") as f:
                code = f.read()
                exec(code, globals())
    except Exception as e:
        print("Auto-exec error:", e)   
    time.sleep(1)

# Hide button
Console('OFF')
PrintImage(img) 
FillRect()
PrintCon()
Console('ON')

# End plugin
plugin.end()
EOF
```

