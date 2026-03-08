# API - Server Side

Most server-side interaction occurs through the files in the `editable/server/` folder.

## Bridge Configuration

In `editable/server/main.lua`, you will find the `EditTable` table, which allows you to modify key server logic without changing the script's main code.

### Key functions to edit:
*   `EditTable.Events.RemoveMoney`: Remove money (e.g., for fines).
*   `EditTable.Events.HasItem`: Check if a player has an item (e.g., for licenses or fines).
*   `EditTable.Events.Query`: Capability to customize the database engine.
*   `EditTable.Events.SendDispatch`: Distribution logic for dispatch alerts for each job.

---

## Server Events

The following event can be triggered from other scripts on the server to integrate actions with MDT.

### `xr-mdt:server:triggerDispatch`
Triggers a dispatch notification sent from the server.

```lua
TriggerServerEvent('xr-mdt:server:triggerDispatch', {
    code = '10-99',
    title = 'Bank Robbery',
    description = 'Armed suspects at the main bank.',
    location = 'Legion Square',
    jobs = {'police'} -- Job groups that will receive the notification
})
```

### `xr-mdt:server:triggerNotification`
Sends a notification to a specific player from MDT.

| Argument | Type | Description |
| :--- | :--- | :--- |
| `text` | string | Text. |
| `type` | string | Type (`'success'`, `'error'`, `'info'`). |
| `length` | number | Time in ms. |

```lua
TriggerNetEvent('xr-mdt:server:triggerNotification', "License granted successfully", 'success', 5000)
```

---

## Conviction Logic (LSPD)

The `LSPD_HandleSentence` function in `server/sv_lspd.lua` handles jail and fine logic. You can edit it if you want to integrate with a different jail script (e.g., `qbx_jail` or `esx_jailer`).
