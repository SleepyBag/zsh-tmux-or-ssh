#!/bin/zsh

function tmux-or-ssh {
    if [[ -z "$FZF_COMMAND" ]]  {
        if ! {which fzf &> /dev/null} {
            echo "fzf is not available!"
            exit 1
        }
        FZF_COMMAND=fzf
    }

    # ssh only when i'm not in a ssh link nor in a tmux session
    if [[ $(is-ssh) == false && -z "$TMUX" ]] {
        sshlist=$(awk '
            $1=="Host"{if(line!=""){print line}; line="ssh: "$0"\011"}
            ($1=="Hostname"||$1=="User"){line=line$0} END{print line}
        ' $HOME/.ssh/config)
    }

    # open a tmux session only when i'm not in a tmux session
    if [[ -z "$TMUX" ]] {
        tmuxlist=$(tmux list-sessions 2> /dev/null | sed "s/^/tmux: /")
    }

    if [[ ! -z "$tmuxlist" || ! -z "$sshlist" ]] {
        # choose one from ssh hosts and tmux sessions
        [[ ! -z "$tmuxlist" ]] && fzflist="${tmuxlist}\n\n${sshlist}" || fzflist=${sshlist}
        fzf_result="$( \
            echo $fzflist | $FZF_COMMAND \
            --print-query \
            --prompt 'Please choose a tmux session or an ssh host > ' \
            --header 'Ctrl-Y: Create a new session'$'\n'" " \
            --bind ctrl-y:print-query \
        )"
        query=$(echo $fzf_result | head -n1)
        choosed=($(echo $fzf_result | tail -n+2))

        # go for it
        if [[ -z "$choosed" && ! -z $query ]] {
            tmux new-session -s $query
        } elif [[ $choosed[1] == tmux: ]] {
            session=$choosed[2]
            tmux attach-session -t $session[1,$(($#session-1))]
        } elif [[ $choosed[1] == ssh: ]] {
            ssh $choosed[3]
        }
    }
}
