# cisco-pt-mcp

Let AI clients control a running Cisco Packet Tracer instance. Open source.

![Demo](https://raw.githubusercontent.com/muhammadbalawal/cisco-pt-mcp/main/docs/demo.gif)

---

## About this fork

This is a fork of [muhammadbalawal/cisco-pt-mcp](https://github.com/muhammadbalawal/cisco-pt-mcp). All credit for the original server and the Packet Tracer extension goes to them.

I'm a CpE student and I was new to networking when I started. For my Networks Lab I had to build a clinic network in Packet Tracer that matched a floor plan. I already knew about MCP and agentic AI, so I looked for a way to let Claude talk to Packet Tracer instead of dragging 130 devices around by hand. This server was exactly that, and it worked: Claude built most of the clinic from the floor plan and I was honestly impressed.

It was also limiting for what I needed, so I changed three things:

- **All 151 device models.** `addDevice` only accepted 11 (a few routers and switches, PC, server, laptop, printer). The list now comes from every device type in the extension's `allDeviceTypes`, so I could place real IP phones (7960), webcams, access points and a TV instead of stand-ins. I placed all of those. The TV (`TV-PT`) has no network port, so it stayed a stand-in, and I never drove the ASA (`5506-X`).
- **Port names.** The cable tools only accepted IOS-style names like `FastEthernet0/1`. Other devices use things like `Port 0` (access point, TV) and `Switch` / `PC` (phone), so the check now accepts any non-empty name and lets Packet Tracer reject what doesn't exist.
- **Duplicate name guard.** Packet Tracer lets you create two devices with the same name, and then every later call by that name is ambiguous. `addDevice` and `renameDevice` now refuse a name that's already on the canvas.

### Still broken

I didn't fix these, and the first two are the ones that hurt most. Most of what tells you a network is really working lives on the devices themselves.

- `configureIosDevice` sends the commands to routers and switches but never reads the output back. A command Packet Tracer rejects still reports success, so I checked configs with pings and port state instead.
- It can't type into a PC's desktop or command prompt, or open a device's Physical tab.
- With an `AccessPoint-PT` or `Webcam` on the canvas, `getNetwork` and `getDeviceInfo` crash. I placed those last.
- The `connections` labels in `getNetwork` are wrong. `in_use` is correct.
- `configurePcIp` only handles `FastEthernet0`. There's no tool to save the file.

### How this was made

I came up with the idea and decided what the project needed. The code was written by Claude Code. I tested it against live Packet Tracer and kept notes on what worked and what didn't.

---

## Install

Two components. Five minutes.

### 1. MCP Server

[Install uv](https://docs.astral.sh/uv/getting-started/installation/) if you don't have it, then register with your client:

**Claude Code**
```sh
claude mcp add cisco-pt-mcp --scope user -- uvx cisco-pt-mcp
```

**Cursor** (`~/.cursor/mcp.json` or `.cursor/mcp.json` per project)
```json
{
  "mcpServers": {
    "cisco-pt-mcp": { "command": "uvx", "args": ["cisco-pt-mcp"] }
  }
}
```

**VS Code** (`.vscode/mcp.json`)
```json
{
  "servers": {
    "cisco-pt-mcp": {
      "type": "stdio",
      "command": "uvx",
      "args": ["cisco-pt-mcp"]
    }
  }
}
```

**Codex CLI** (`~/.codex/config.toml`)
```toml
[mcp_servers.cisco-pt-mcp]
command = "uvx"
args = ["cisco-pt-mcp"]
```

### 2. Packet Tracer Extension

1. Open Packet Tracer.
2. Go to **Extensions → Scripting → Configure PT Script Modules…**
3. Click **Add** and select [`cisco-pt-mcp.pts`](extension/cisco-pt-mcp.pts).
4. Restart Packet Tracer.

### 3. Run

Click **Extensions → Packet Tracer MCP** to open the bridge window, then start your MCP client. The status pill flips to `connected` within a second. Start prompting.

---

## Tools

| Tool | Description |
|---|---|
| `addDevice` | Drop a router, switch, or PC onto the canvas |
| `addModule` | Install an interface module into a device slot |
| `addLink` | Connect two devices with a cable |
| `removeDevice` | Delete one or more devices |
| `removeLink` | Delete one or more cables |
| `renameDevice` | Rename an existing device |
| `moveDevice` | Reposition a device on the canvas |
| `setPower` | Power a device on or off |
| `configurePcIp` | Set IP, subnet, gateway, DNS, or enable DHCP on a PC |
| `configureIosDevice` | Run IOS CLI commands on a router or switch |
| `getNetwork` | Snapshot of all devices, interfaces, and cables |
| `getDeviceInfo` | Detailed view of a single device |
| `setSimulationMode` | Switch between simulation and realtime mode |
| `getSimulationStatus` | Query simulation state and frame count |
| `stepSimulation` | Step the simulation forward, backward, or reset |
| `sendPdu` | Add an ICMP ping PDU between two devices |
| `getPduResults` | Read PDU outcomes after stepping the simulation |
| `getCommandLog` | Read the IOS command history logged by Packet Tracer |

---

## License

MIT
