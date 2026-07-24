# Running `day0_provision.py` — Student Guide

This guide walks you through running the day-0 provisioning script from your **native host OS** (Windows, Mac, or Linux). The console push needs direct USB access to your serial adapter, which a VM does not have without USB passthrough.

Labs that use this tool tell you which `.zip` to download and which config
file (e.g. `day0-lab12.yaml`) to pass in. This guide only covers the
mechanics of running the script itself — follow it once you've downloaded
and extracted the lab's zip file to your Desktop.

---

## 1. Extract the zip on your Desktop

Right-click the `.zip` file on your Desktop and choose **Extract All...**, or
from PowerShell:

```powershell
cd Desktop
Expand-Archive -Path lab12.zip -DestinationPath lab12
```

This creates a folder (e.g. `Desktop\lab12\`) containing `day0_provision.py`,
its templates, the `.yaml` config file(s), and the other tools the lab needs.

## 2. `cd` into that folder before doing anything else

Every command below assumes your terminal's current directory **is** the
extracted folder — not your Desktop, not your home directory. If you skip
this step, `day0_provision.py` will not find its config or template files
and will fail with a "not found" error that looks like a bug but isn't.

```powershell
cd Desktop\lab12
```

Confirm you're in the right place before continuing:

```powershell
dir
```

You should see `day0_provision.py`, one or more `.yaml` files, and a
`templates\` folder listed. If you don't, you are in the wrong directory —
`cd` again until `dir` shows these files.

## 3. Install requirements (first time only)

```powershell
pip install -r requirements.txt
```

This installs `pyyaml` (always required) and `pyserial` (only needed for a
live push — dry runs work without it). If you already installed these for
an earlier lab, you can skip this step.

## 4. Preview the config before pushing anything

Just leave `--port` off. Omitting `--port` **is** the dry run — it renders
the config and prints it, and sends nothing to any device:

```powershell
python day0_provision.py --config day0-lab12.yaml --u <your-U> --username <your-username>
```

The script shows a numbered menu listing the routers you can configure — pick whichever one you want to preview. Read the output.
Confirm your U number and username were substituted correctly and nothing
looks wrong before you move to a live push.

## 5. Push for real, one device at a time

Run the script **once per device** — CORE, then EDGE — each time plugged
into that device's console port:

```powershell
python day0_provision.py --config day0-lab12.yaml --u <your-U> --username <your-username> --port <your-serial-port>
```

Replace `<your-serial-port>` with your console adapter's port (e.g.
`COM3` on Windows, `/dev/ttyUSB0` on Linux, `/dev/tty.usbserial-XXXX` on
Mac — check Device Manager on Windows if you're not sure which COM port).

The script shows a menu — pick the device you're physically plugged into
right now (CORE or EDGE) — then asks you to type that device's name to
confirm before it sends anything. Run the command again for the other
device once you've moved your console cable.

## 6. What "success" looks like

When the push finishes, the script prints a line telling you the device is
provisioned and should be SSH-reachable. If it also ran verify checks
(most configs include some), you'll see a `[✔]` or `[✘]` next to each one —
all `[✔]` means the push is confirmed good.

---

## Command reference

```text
Required:
  --config PATH        the provisioning YAML for this lab (e.g. day0-lab12.yaml)

Your identity:
  --u VALUE             your U number
  --username VALUE      your username

Device:
  --device DEVICE       CORE or EDGE — optional. Leave it off and the script
                        shows you a menu to pick from; you don't need to
                        type a device name on the command line.

Console:
  --port PORT           serial port (COM3, /dev/ttyUSB0, ...). Leave it off
                        for a dry run — this is the default, no separate
                        flag needed.
  --dry-run             an explicit way to force a dry run even if you did
                        pass --port; you won't normally need this
  --yes                 skip the confirmation prompt
  --quiet               don't echo each line as it is sent
```

There is no `--self` flag — your identity always comes from `--u` and
`--username`, whether you're provisioning your own pod or (for staff use)
someone else's.

---

## Troubleshooting

**"config not found" / "template not found"** — you are not in the
extracted lab folder. Run `dir` (or `ls`) and confirm `day0_provision.py`
and the `.yaml` file are in your current directory; `cd` to the right place
and try again.

**"template contains unresolved variables"** — you omitted `--u` or
`--username` and the template needs one of them. Add the missing flag.

**"pyserial is required for live pushes"** — you skipped step 3, or only
installed it after already trying a live push. Run
`pip install -r requirements.txt` and try again.

**Push runs but SSH doesn't come up afterward** — re-run the same command
without `--port` to double check the rendered config looks right, then
retry the live push. If it still fails, flag it to your instructor rather
than repeatedly re-pushing — repeated pushes to a partially-configured
device can leave it in a confusing state.

**Wrong COM port / permission errors on the serial port** — on Windows,
check Device Manager → Ports (COM & LPT) for the correct COM number; on
Mac/Linux, run `ls /dev/tty.*` or `ls /dev/ttyUSB*` to list available
ports, and make sure no other terminal or app (like PuTTY) already has that
port open.
