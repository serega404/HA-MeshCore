# HA-MeshCore

Репозиторий с моими автоматизациями для аддона MeshCore в HomeAssistant

## MeshCore to Telegram topics

```yaml
alias: MeshCore to Telegram topics
triggers:
  - event_type: meshcore_message
    event_data:
      message_type: channel
    trigger: event
conditions:
  - condition: template
    value_template: "{{ tg_thread is not none }}"
  - condition: template
    value_template: "{{ mesh_sender | lower not in ignored_mesh_senders }}"
actions:
  - action: telegram_bot.send_message
    data:
      entity_id:
        - notify.s_home_bot_meshcore
      message_thread_id: "{{ tg_thread }}"
      parse_mode: plain_text
      message: "{{ mesh_sender }}: {{ mesh_message }}"
mode: queued
variables:
  mesh_channel: "{{ trigger.event.data.channel_idx | int(-1) }}"
  mesh_sender: "{{ trigger.event.data.sender_name | default('MeshCore', true) }}"
  mesh_message: "{{ trigger.event.data.message | default('', true) }}"
  ignored_mesh_senders:
    - owner name lower case
  tg_thread_map:
    "0": 1
    "2": 6
    "3": 4
  tg_thread: "{{ tg_thread_map.get(mesh_channel | string) }}"
```

### Что менять MeshCore->TG

Нужно заменить `owner name lower case` на имя отправителя MeshCore в нижнем регистре, которого вы хотите игнорировать (например, самого себя, если не хотите получать свои сообщения в Telegram). Также нужно настроить `tg_thread_map`, указав соответствия между каналами MeshCore и топиками Telegram.

Так же потребуется заменить `notify.s_home_bot_meshcore` на ваш идентификатор уведомлений Telegram в Home Assistant, если он отличается.

## Telegram topics to MeshCore

```yaml
alias: Telegram topics to MeshCore
description: ""
triggers:
  - trigger: state
    entity_id:
      - event.s_home_bot_update_event
conditions:
  - condition: state
    entity_id: event.s_home_bot_update_event
    attribute: event_type
    state: telegram_text
  - condition: template
    value_template: "{{ mesh_channel is not none }}"
  - condition: template
    value_template: "{{ tg_text | trim | length > 0 }}"
actions:
  - action: meshcore.send_channel_message
    data:
      channel_idx: "{{ mesh_channel | int }}"
      message: "{{ tg_sender }}: {{ tg_text }}"
mode: queued
variables:
  tg_thread: "{{ trigger.to_state.attributes.message_thread_id | int(-1) }}"
  tg_text: "{{ trigger.to_state.attributes.text | default('', true) }}"
  tg_sender: "{{ trigger.to_state.attributes.from_first | default('Telegram', true) }}"
  tg_thread_to_mesh_map:
    "1": 0
    "4": 3
    "6": 2
  mesh_channel: "{{ tg_thread_to_mesh_map.get(tg_thread | string) }}"
```

### Что менять TG->MeshCore

Нужно настроить `tg_thread_to_mesh_map`, указав соответствия между топиками Telegram и каналами MeshCore. Также убедитесь, что `event.s_home_bot_update_event` соответствует вашему идентификатору события обновления Telegram в Home Assistant, если он отличается.
