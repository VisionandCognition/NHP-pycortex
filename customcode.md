## Alterations to the original pycortex code
Here we document the alterations to the original pycortex code we needed to make in order to make things compatible with our NHP-Freesurfer data.

### In `cortex/freesurfer.py`     
The `import_subj` function expects components that we don't have for NHP's. We copied it to `import_subjnhp` and took out the parts that don't work.

```python   
def import_subjnhp(fs_subject, cx_subject=None, freesurfer_subject_dir=None, whitematter_surf='smoothwm'):
    """Imports a subject from freesurfer

    This will overwrite (after giving a warning and an option to continue) the 
    pre-existing subject, including all blender cuts, masks, transforms, etc., and
    re-generate surface info files (curvature, sulcal depth, thickness) stored in 
    the surfinfo/ folder for the subject. All cached files for the subject will be 
    deleted. 

    Parameters
    ----------
    fs_subject : string
        Freesurfer subject name
    cx_subject : string, optional
        Pycortex subject name (These variable names should be changed). By default uses
        the same name as the freesurfer subject. Best to stick to that convention, if 
        possible (your life will go more smoothly.) This optional kwarg is for edge cases. 
    freesurfer_subject_dir : string, optional
        Freesurfer subject directory to pull data from. By default uses the directory
        given by the environment variable $SUBJECTS_DIR.
    whitematter_surf : string, optional
        Which whitematter surface to import as 'wm'. By default uses 'smoothwm', but that
        surface is smoothed and may not be appropriate. A good alternative is 'white'.
    """
    if cx_subject is None:
        cx_subject = fs_subject
    # Create and/or replace extant subject. Throws a warning that this will happen.
    database.db.make_subj(cx_subject)

    import nibabel
    surfs = os.path.join(database.default_filestore, cx_subject, "surfaces", "{name}_{hemi}.gii")
    anats = os.path.join(database.default_filestore, cx_subject, "anatomicals", "{name}.nii.gz")
    surfinfo = os.path.join(database.default_filestore, cx_subject, "surface-info", "{name}.npz")
    if freesurfer_subject_dir is None:
        freesurfer_subject_dir = os.environ['SUBJECTS_DIR']
    fspath = os.path.join(freesurfer_subject_dir, fs_subject, 'mri')
    curvs = os.path.join(freesurfer_subject_dir, fs_subject, 'surf', '{hemi}.{name}')

    #import anatomicals
    for fsname, name in dict(T1="raw", brainmask="brainmask", wm="whitematter").items():
        path = os.path.join(fspath, "{fsname}.mgz").format(fsname=fsname)
        out = anats.format(subj=cx_subject, name=name)
        cmd = "mri_convert {path} {out}".format(path=path, out=out)
        sp.check_output(shlex.split(cmd))

    # (Re-)Make the fiducial files
    # NOTE: these are IN THE FREESURFER $SUBJECTS_DIR !! which can cause confusion.
    make_fiducial(fs_subject, freesurfer_subject_dir=freesurfer_subject_dir)

    # Freesurfer uses FOV/2 for center, let's set the surfaces to use the
    # magnet isocenter ---> WHY?
    trans = nibabel.load(out).get_affine()[:3, -1]
    surfmove = trans - np.sign(trans) * [128, 128, 128]

    from . import formats
    for fsname, name in [(whitematter_surf, "wm"), ('pial', "pia"), ('inflated', "inflated")]:
        for hemi in ("lh", "rh"):
            pts, polys, _ = get_surf(fs_subject, hemi, fsname, freesurfer_subject_dir=freesurfer_subject_dir)
            fname = str(surfs.format(subj=cx_subject, name=name, hemi=hemi))
            formats.write_gii(fname, pts=pts + surfmove, polys=polys)
            #formats.write_gii(fname, pts=pts, polys=polys)

    for curv, info in dict(sulc="sulcaldepth", thickness="thickness", curv="curvature").items():
        lh, rh = [parse_curv(curvs.format(hemi=hemi, name=curv)) for hemi in ['lh', 'rh']]
        np.savez(surfinfo.format(subj=cx_subject, name=info), left=-lh, right=-rh)

    database.db = database.Database()
```

### In `cortex/svgoverlay.py`    
Around line `268` we changed the call to inkscape to make the matplolib plotting work on mac osx where we need a 64bit beta version of Inkscape that takes commmand line arguments in a different format. To facilitate this we now import `sys` and check what operating system we're on. This means **all** Mac OS versions will use the new call.

```python
import sys
if sys.platform == 'darwin':
    cmd = "{inkscape_cmd} -z -h {height} --export-file {outfile} /dev/stdin" # inkscape beta
else:   
    cmd = "{inkscape_cmd} -z -h {height} --e {outfile} /dev/stdin" # older inkscape
```    


