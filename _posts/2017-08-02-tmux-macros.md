---
layout: post
title:  "Макросы для tmux"
date:   2017-08-02 00:00:00
---

Недавно я обнаружил, что я постоянно создаю в tmux всего два типа рабочих мест:

* Основная вкладка, разделённая на 4 панели, в одном из которых запущен htop:
  [![](/assets/images/tmux-macros/tdef.png)](/assets/images/tmux-macros/tdef.png)
* Вкладка для работы с программным кодом, 3 четверти которой занимает текстовый редактор, а оставшееся место поделено на 2 панели:
  [![](/assets/images/tmux-macros/tide.png)](/assets/images/tmux-macros/tide.png)

Чтобы постоянно не создавать их руками, я решил сделать макросы, автоматически генерирующие такие разбиения терминала. Гугление не дало вариантов, которые меня бы устроили: зачастую люди использовали для подобных задач фреймворк [tmuxinator](https://github.com/tmuxinator/tmuxinator), но мне не хотелось тащить к себе громоздкую зависимость просто для того, чтобы создавать панельки в терминале.

Поэтому я пришёл к более легковесному решению. В репозиторий с [дотфайлами](/2017/08/01/dotfiles) добавляются два скрипта:

```sh
#!/usr/bin/zsh
tmux split-window -h -c "#{pane_current_path}"
tmux split-window -v -c "#{pane_current_path}"
tmux select-pane -t 0
tmux split-window -v -c "#{pane_current_path}"
tmux select-pane -t 3
tmux send-keys "htop" C-m
tmux select-pane -t 0
```

```sh
#!/usr/bin/zsh
tmux split-window -h -p 25 -c "#{pane_current_path}"
tmux split-window -v -c "#{pane_current_path}"
tmux select-pane -t 0
```

Как можно увидеть, это просто создание панелей стандартными средствами tmux.

Далее в список алиасов добавляются две строчки:

```sh
alias tdef='~/.dotfiles/tmux/default-session.sh'
alias tide='~/.dotfiles/tmux/ide-session.sh'
```

Теперь при открытии новой вкладки в tmux я могу просто ввести `tide` и мгновенно получить нужное мне разбиение на панели.