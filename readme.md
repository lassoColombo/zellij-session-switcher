# zellij-session-switcher

Switch zellij session from the shell, with one command that does the right thing on both sides
of the fence: inside a session it moves the current client with `zellij action switch-session`,
outside it attaches with `zellij attach`.

```nu
zellij-session-switcher            # pick a session
zellij-session-switcher home       # switch to `home`
zellij-session-switcher ho         # switch to `home` too, if nothing else matches
zellij-session-switcher ls         # the sessions as a table
```

The argument is matched against the session names: an exact name wins, otherwise it is used as a
regex, and if it matches more than one session you are asked to choose. Pass nothing and you are
asked straight away. A name that matches nothing is an error - this switches between sessions,
it never creates one.

Autocompletion covers the same ground: Tab offers the live sessions, each with its state and age
as description, the current one in green and the exited ones dimmed. Descriptions need a menu
with room for them, so bind the ide menu if you want to see them on Tab.

`ls` returns what the switcher works on - `name`, `created` (a real duration, so you can sort by
it), `current` and `exited`:

```nu
zellij-session-switcher ls | where not exited | sort-by created
```

## Configure the picker

Choosing a session uses Nushell's built-in `input list` by default. Set
`$env.zellij_session_switcher_config.picker` to swap the engine. A picker is a closure from a
list of sessions to one session:

```
list<record<name: string, created: duration, current: bool, exited: bool>> -> record | null
```

`zellij-session-switcher ls` returns exactly those records, so a picker is a command you can run
by hand against real data:

```nu
$env.zellij_session_switcher_config = {picker: {|| $in | where not exited | first }}
```

Returning nothing cancels the switch.

## Installation

```nu
# clone into one of your NU_LIB_DIRS
let dest = [($env.NU_LIB_DIRS | first) zellij-session-switcher] | path join
git clone git@github.com:lassoColombo/zellij-session-switcher.git $dest

# use the module, and give it a shorter name while you are at it
use zellij-session-switcher
alias zs = zellij-session-switcher
```

Import the module, not its contents: `use zellij-session-switcher *` would put `ls` in your
scope on top of the built-in one.
