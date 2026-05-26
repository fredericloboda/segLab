# SegLab

SegLab is a desktop application for brain lesion segmentation on T1-weighted MRI with manual mask creation, gold-mask comparison, Dice/IoU-based evaluation, progress tracking, and offline teaching-package import.

## What SegLab does

SegLab is built for offline lesion-segmentation training.

It allows users to:

* work on built-in cases packaged inside the app
* import offline teaching packages (`.seglabpack`)
* open cases in ITK-SNAP
* create or edit lesion masks
* compare masks against gold-standard masks
* score attempts with Dice and IoU
* review per-case and overall progress

No server is required.

---

## Repository structure

Typical project structure:

```text
seglab/
├── README.md
├── startt_trainer.py
├── build_win.ps1
├── build_mac.sh
├── ui/
├── modules/
├── resources/
│   ├── bundled_cases/
│   ├── bundled_materials/
│   ├── icon.png
│   └── icon.ico
├── build/
├── dist/
└── ...
```

Important folders:

* `resources/bundled_cases/` → permanent built-in cases
* `resources/bundled_materials/` → optional built-in teaching files
* `dist/` → built app output

---

## Build the app

### Windows

From the project folder, run:

```powershell
cd "C:\path\to\seglab"
powershell -ExecutionPolicy Bypass -File .\build_win.ps1
```

If the app builds but the zip is not created automatically, run:

```powershell
Compress-Archive -Path ".\dist\SegLab\*" -DestinationPath ".\dist\SegLab_windows_x64.zip" -Force
```

Expected output:

```text
dist/SegLab_windows_x64.zip
```

Example:

```powershell
cd "C:\Users\YourName\Downloads\seglab"
powershell -ExecutionPolicy Bypass -File .\build_win.ps1
```

If Python is not found on Windows but `py` works, you can build manually from `cmd` like this:

```bat
cd /d "C:\Users\YourName\Downloads\seglab"
py -3 -m venv ".venv_build"
".venv_build\Scripts\python.exe" -m pip install -U pip
".venv_build\Scripts\python.exe" -m pip install PySide6 numpy nibabel pyinstaller requests packaging certifi pillow
".venv_build\Scripts\python.exe" -m pip uninstall -y PyQt5 PyQt6
if not exist "resources\icon.ico" ".venv_build\Scripts\python.exe" -c "from PIL import Image; img=Image.open(r'resources\icon.png').convert('RGBA'); img.save(r'resources\icon.ico', sizes=[(16,16),(32,32),(48,48),(64,64),(128,128),(256,256)]); print('Wrote resources/icon.ico')"
rmdir /s /q build 2>nul
rmdir /s /q dist 2>nul
del /q *.spec 2>nul
".venv_build\Scripts\python.exe" -m PyInstaller --noconfirm --clean --windowed --name SegLab --exclude-module PyQt5 --exclude-module PyQt6 --add-data "resources;resources" --icon ".\resources\icon.ico" startt_trainer.py
powershell -Command "Compress-Archive -Path 'C:\Users\YourName\Downloads\seglab\dist\SegLab\*' -DestinationPath 'C:\Users\YourName\Downloads\seglab\dist\SegLab_windows_x64.zip' -Force"
dir "C:\Users\YourName\Downloads\seglab\dist\SegLab_windows_x64.zip"
```

### macOS

From the project folder, run:

```bash
cd "/path/to/seglab"
bash build_mac.sh
```

To zip the app:

```bash
cd dist
ditto -c -k --sequesterRsrc --keepParent "SegLab.app" "SegLab_macOS.app.zip"
```

Expected output:

```text
dist/SegLab.app
dist/SegLab_macOS.app.zip
```

Example:

```bash
cd "/Users/YourName/Downloads/seglab"
bash build_mac.sh
```

If macOS blocks the app after build or download, remove quarantine:

```bash
xattr -dr com.apple.quarantine "/path/to/SegLab.app"
```

If the app icon bounces but no window appears, run the executable directly to see the error:

```bash
cd "/path/to/SegLab.app/Contents/MacOS"
./SegLab
```

---

## Built-in cases

Permanent training cases can be packaged directly into the app.

Put one folder per case into:

`resources/bundled_cases/`

Each case folder should contain:

* `t1.nii.gz`
* `gold.nii.gz`
* `case.json`

Example:

```text
resources/bundled_cases/builtin_01/
  t1.nii.gz
  gold.nii.gz
  case.json

resources/bundled_cases/builtin_02/
  t1.nii.gz
  gold.nii.gz
  case.json
```

Minimal `case.json`:

```json
{
  "case_id": "builtin_01",
  "label": "Built-in 01"
}
```

After adding built-in cases, rebuild the app.

---

## Built-in materials

Optional teaching files can be packaged into the app by placing them inside:

`resources/bundled_materials/`

These can be PDF, PPTX, DOCX, TXT, or other local teaching files.

Example:

```text
resources/bundled_materials/
  Introduction.pdf
  Week1_Slides.pptx
  Anatomy_Notes.pdf
```

After adding materials, rebuild the app.

---

## Offline teaching packages

SegLab supports offline package distribution through `.seglabpack`.

This is the intended workflow when a teacher wants to prepare cases once and send them to students without any server, SMB path, classroom sync, or app rebuild for every teaching session.

### Teacher workflow

1. Prepare one folder containing case folders.
2. Optionally prepare one folder containing teaching materials.
3. Open SegLab.
4. Go to **Teacher**.
5. Click **Add cases…**
6. Optionally click **Add materials…**
7. Click **Preview package**
8. Click **Create package**
9. Save one `.seglabpack`.
10. Send that `.seglabpack` to students.

A valid case folder contains:

```text
Case_01/
  t1.nii.gz
  gold.nii.gz
  case.json
```

A folder containing multiple cases can look like:

```text
Cases/
  Case_01/
    t1.nii.gz
    gold.nii.gz
    case.json
  Case_02/
    t1.nii.gz
    gold.nii.gz
    case.json
  Case_03/
    t1.nii.gz
    gold.nii.gz
    case.json
```

Select the top-level `Cases/` folder in **Add cases…**.

### Student workflow

1. Open SegLab.
2. Go to **Practice**.
3. Click **Import pack**.
4. Select the `.seglabpack`.
5. The cases are installed locally and appear in Practice.

Imported packs are intended to remain locally available after import.

Multiple packs should be additive unless duplicate case IDs require overwrite handling.

---

## Segmentation workflow

In **Practice**:

1. Select a built-in or imported case.
2. Click **Open**.
3. The case opens in **ITK-SNAP**.
4. Create or edit the lesion mask.
5. Save the mask.
6. Return to SegLab.
7. Use:

   * **Evaluate**
   * **Compare**
   * **History**
   * **Progress**

Current evaluation includes:

* Dice
* IoU

Current progress tracking includes:

* first attempt
* latest attempt
* best attempt
* attempt history
* overall case progress

---

## Synthetic test workflow

You can test SegLab using synthetic cases before using real data.

### macOS synthetic case generation

```bash
python3 -m pip install -U nibabel numpy
python3 - <<'PY'
import os, json, numpy as np, nibabel as nib
root = "/tmp/SegLabSynthetic/cases"
os.makedirs(root, exist_ok=True)
cases = [
    ("case_demo_001",(28,30,14),8),
    ("case_demo_002",(36,34,18),7),
    ("case_demo_003",(24,40,16),6),
    ("case_demo_004",(40,24,15),9),
    ("case_demo_005",(20,22,20),5),
]
affine = np.eye(4)
shape = (64,64,32)
xx,yy,zz = np.meshgrid(np.arange(shape[0]), np.arange(shape[1]), np.arange(shape[2]), indexing="ij")

for cid, ctr, r in cases:
    p = os.path.join(root, cid)
    os.makedirs(p, exist_ok=True)
    img = ((xx-shape[0]/2)**2 + (yy-shape[1]/2)**2 + (zz-shape[2]/2)**2).astype(np.float32)
    img = (img-img.min())/(img.max()-img.min())
    img = 1000*(1-img) + np.random.normal(0, 8, shape).astype(np.float32)
    mask = (((xx-ctr[0])**2 + (yy-ctr[1])**2 + (zz-ctr[2])**2) <= r*r).astype(np.uint8)
    nib.save(nib.Nifti1Image(img, affine), os.path.join(p, "t1.nii.gz"))
    nib.save(nib.Nifti1Image(mask, affine), os.path.join(p, "gold.nii.gz"))
    with open(os.path.join(p, "case.json"), "w", encoding="utf-8") as f:
        json.dump({"case_id": cid, "label": cid}, f, indent=2)

print("Created synthetic cases in", root)
PY
```

### Windows synthetic case generation

```bat
py -3 -m pip install -U nibabel numpy
py -3 -c "import os, json, numpy as np, nibabel as nib; root=r'C:\Temp\SegLabSynthetic\cases'; os.makedirs(root, exist_ok=True); cases=[('case_demo_001',(28,30,14),8),('case_demo_002',(36,34,18),7),('case_demo_003',(24,40,16),6),('case_demo_004',(40,24,15),9),('case_demo_005',(20,22,20),5)]; affine=np.eye(4); shape=(64,64,32); xx,yy,zz=np.meshgrid(np.arange(shape[0]),np.arange(shape[1]),np.arange(shape[2]), indexing='ij');
for cid,ctr,r in cases:
 p=os.path.join(root,cid); os.makedirs(p, exist_ok=True)
 img=((xx-shape[0]/2)**2+(yy-shape[1]/2)**2+(zz-shape[2]/2)**2).astype(np.float32)
 img=(img-img.min())/(img.max()-img.min()); img=1000*(1-img)
 img=img + np.random.normal(0, 8, shape).astype(np.float32)
 mask=(((xx-ctr[0])**2+(yy-ctr[1])**2+(zz-ctr[2])**2)<=r*r).astype(np.uint8)
 nib.save(nib.Nifti1Image(img, affine), os.path.join(p,'t1.nii.gz'))
 nib.save(nib.Nifti1Image(mask, affine), os.path.join(p,'gold.nii.gz'))
 open(os.path.join(p,'case.json'),'w',encoding='utf-8').write(json.dumps({'case_id':cid,'label':cid}, indent=2))
print('Created synthetic cases in', root)"
```

### Testing the package workflow with synthetic cases

Teacher:

* **Add cases…**
* choose the synthetic `cases` folder
* **Preview package**
* **Create package**
* save one `.seglabpack`

Student:

* **Import pack**
* select the generated `.seglabpack`

Then test:

* **Open**
* **Evaluate**
* **Compare**
* **History**
* **Progress**

---

## Recommended deployment model

The simplest deployment model is:

* one fixed SegLab app
* multiple `.seglabpack` teaching packages

This avoids:

* repeated rebuilding for every class
* SMB setup
* server infrastructure
* classroom sync complexity

---

## Notes

* SegLab is designed as an offline segmentation-training workflow.
* No server is required for built-in cases or content-pack use.
* Built-in cases and imported cases can coexist.
* Imported packs are intended to install cases locally for later reuse.
* SegLab currently focuses on T1-weighted brain MRI lesion segmentation, manual annotation, gold-mask comparison, and progress evaluation.
