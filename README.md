# YasaraVSCode
Execute Yasara commands from VSCode


## installation
```
# create the file to which current selection is written
touch "$HOME/.vscode/save_hl.txt"

# shell script that writes the file
echo \
'#!/bin/bash
pbpaste > ~/.vscode/save_hl.txt' \
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

file_to_watch = os.path.expanduser("~/.vscode/save_hl.txt")
last_mtime = 0

while True:
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

# End plugin
plugin.end()
EOF
```

