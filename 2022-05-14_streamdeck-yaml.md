---
title: streamdeck-yaml — Controlling Home Assistant with an Elgato Stream Deck
published: 2022-05-14T21:00:00+02:00
mainImg: 05.jpg
mainImgOffset: 22
tags:
  - programming
github: https://github.com/rookies/streamdeck-yaml
...
I love controlling stuff with Home Assistant, but having to open the app on my phone each time I
want to switch on a light gets annoying really quickly. As I'm also not a big fan of yelling  at
personal assistants like Alexa or Siri, I wanted to control everything conveniently with physical
buttons — and the Elgato Stream Decks are just great for that. So I wrote some software that runs
on my Raspberry Pi and connects a Stream Deck with Home Assistant.

First, let's have a look at what you can do with it.

![My main menu with two sub-menus (and space for more)](01.jpg)

You can define sub-menus, e.g. for all your lights and media devices or for different rooms.

![After entering the “Lights” sub-menu ...](02.jpg)

Buttons for controlling the lights are possible ...

![... and switching on the wall light](03.jpg)

... and they change their icon and color depending on the state of the light.

![Back to the main menu and into “Media” ...](04.jpg)

Switches are supported ...

![... and switching on the speakers](05.jpg)

... and they also change their appearance. The “Cinema” button activates a Home Assistant
script.

If you like the idea, why not give it a try? The functionality is at the moment basically limited
to the things I'm personally using, but I'm always open for pull requests :)

## Configuration

Everything is configured in a single YAML file. This is my config for the menus you saw in
the photos above:

    #!yaml
    ---

    # Here you define sub-menus and actions for the keys of the Stream Deck:
    keys:
      # The first key opens a sub-menu:
      - kind: SubMenu
        values:
          icon: lightbulb-multiple
          # ... which is called “Lights”:
          title: Lights
          # ... and contains a new set of keys:
          keys:
            # ... like a button that toggles the state of a light:
            - kind: HomeAssistantToggle
              backend: ha
              values:
                title: Bookshelf
                entity_id: light.bedroom_bookshelf_lamp
            - kind: HomeAssistantToggle
              backend: ha
              values:
                title: Deco
                entity_id: light.bedroom_decoration_lamps
            - kind: HomeAssistantToggle
              backend: ha
              values:
                title: Wall
                entity_id: light.bedroom_window_wall_lamp
            - null
            - null
            # ... and a back button that returns to the previous menu:
            - kind: BackButton
      # The second key opens another sub-menu:
      - kind: SubMenu
        values:
          # ... this time called “Media”:
          title: Media
          icon: projector
          keys:
            # ... which allows toggling the state of switches:
            - kind: HomeAssistantToggle
              backend: ha
              values:
                title: Audio
                entity_id: switch.bedroom_audio
            - kind: HomeAssistantToggle
              backend: ha
              values:
                title: Speakers
                entity_id: switch.bedroom_main_speakers
            - kind: HomeAssistantToggle
              backend: ha
              values:
                title: Projector
                entity_id: switch.bedroom_projector
            - kind: HomeAssistantToggle
              backend: ha
              values:
                title: herschel
                entity_id: switch.herschel
            # ... and the activation of a script:
            - kind: HomeAssistantScript
              backend: ha
              values:
                icon: theater
                title: Cinema
                entity_id: script.cinema_prepare
            - kind: BackButton

    # You also need to define a frontend that displays the keys and accepts
    # keypresses. Currently, ElgatoFrontend for Elgato Stream Decks and
    # GtkFrontend for development purposes are implemented.
    frontend:
      kind: ElgatoFrontend
      rows: 2
      columns: 3
      # The timeout defines the amount of time after which the display is disabled,
      # it will be activated again if any key is pressed.
      timeout: 300  # seconds, i.e. 5 minutes

    # The style is customizable with a custom font, fontsize, and padding at the borders of
    # the key images:
    style:
      font: /usr/share/fonts/TTF/OpenSans-Bold.ttf
      max_fontsize: 15
      padding: 7

    # Last but not least, you need to define backends for the keys. Currently only Home
    # Assistant is supported, but you can add multiple instances here and reference them
    # by their name in the `backend` parameter of the keys.
    backends:
      ha:
        kind: HomeAssistantBackend
        values:
          url: "wss://<your Home Assistant URL>/api/websocket"
          token: "<your Home Assistant access token>"
