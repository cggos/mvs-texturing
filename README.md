MVS-Texturing
--------------------------------------------------------------------------------

Welcome to our project that textures 3D reconstructions from images.
This project focuses on 3D reconstructions generated using structure from
motion and multi-view stereo techniques, however, it is not limited to this
setting.

The algorithm was published in Sept. 2014 on the
*European Conference on Computer Vision*. Please refer to our project website
(http://www.gcc.tu-darmstadt.de/home/proj/texrecon/)
for the paper and further information.

*Please be aware that while the interface of the `texrecon` application is
relatively stable the interface of the `tex` library is currently subject to
frequent changes.*


Dependencies
--------------------------------------------------------------------------------

The code and the build system have the following prerequisites:

- cmake (>= 3.1)
- git
- make
- gcc (>= 5.0.0) or a compatible compiler
- libpng, libjpg, libtiff, libtbb


Furthermore the build system automatically downloads and compiles the following
dependencies (so there is nothing you need to do here):

- rayint
    https://github.com/nmoehrle/rayint
- Eigen
    http://eigen.tuxfamily.org
- Multi-View Environment
    http://www.gcc.tu-darmstadt.de/home/proj/mve
- mapMAP
    http://www.gcc.tu-darmstadt.de/home/proj/mapmap


Compilation ![Build Status](https://travis-ci.org/nmoehrle/mvs-texturing.svg)
--------------------------------------------------------------------------------

1.  `git clone https://github.com/nmoehrle/mvs-texturing.git`
2.  `cd mvs-texturing`
3.  `mkdir build && cd build && cmake ..`
4.  `make` (or `make -j` for parallel compilation)

If something goes wrong during compilation you should check the output of the
cmake step. CMake checks all dependencies and reports if anything is missing.

If you think that there is some problem with the build process on our side
please tell us.

If you are trying to compile this under windows (which should be possible but
we haven't checked it) and you feel like we should make minor fixes to support
this better, you can also tell us.


Execution
--------------------------------------------------------------------------------

As input our algorithm requires a triangulated 3D model and images that are
registered against this model. One way to obtain this is to:
*   import images, infer camera parameters and reconstruct depth maps
    using the [Multi-View Environment]
    (http://www.gcc.tu-darmstadt.de/home/proj/mve/),
    and
*   fuse these depth maps into a combined 3D model using the
    [Floating Scale Surface Reconstruction]
    (http://www.gcc.tu-darmstadt.de/home/proj/fssr/)
    algorithm.

```sh
# Usage: ./apps/texrecon/texrecon [options] IN_SCENE IN_MESH OUT_PREFIX

cd build
./apps/texrecon/texrecon --data_term=area --keep_unseen_faces ../../data/textured_mesh_scene/face000/keyFrames/ ../../data/textured_mesh_scene/face000/mesh/meshRefine.ply ../../data/out/face000
```

* IN_MESH
  - Triangle Mesh Model

* IN_SCENE (dir)
  - keyFrame00xx.CAM
    ```cpp
    struct CameraInfo {
      public:
      /* Intrinsic camera parameters. */

      /** Focal length. */
      float flen;
      /** Principal point in x- and y-direction. */
      float ppoint[2];
      /** Pixel aspect ratio pixel_width / pixel_height. */
      float paspect;
      /** Image distortion parameters. */
      float dist[2];

      /* Extrinsic camera parameters. */

      /** Camera translation vector. Camera position p = -ROT^T * trans. */
      float trans[3];
      /** Camera rotation which transforms from world to cam. */
      float rot[9];
    };

    CameraInfo camera;
    camera.flen = mParams.fx / mParams.cols;
    camera.ppoint[0] = mParams.cx / mParams.cols;
    camera.ppoint[1] = mParams.cy / mParams.rows;
    camera.dist[0] = 0.0;
    camera.dist[1] = 0.0;
    camera.paspect = 1.0; // 1.0 // mParams.cols / mParams.rows
    // keyFrameInfo.poseAffinetInv: Tcw (Tcnc0)
    // rotation, row-major
    camera.rot[0] = keyFrameInfo.poseAffinetInv.matrix.val[0];
    camera.rot[1] = keyFrameInfo.poseAffinetInv.matrix.val[1];
    camera.rot[2] = keyFrameInfo.poseAffinetInv.matrix.val[2];
    camera.rot[3] = keyFrameInfo.poseAffinetInv.matrix.val[4];
    camera.rot[4] = keyFrameInfo.poseAffinetInv.matrix.val[5];
    camera.rot[5] = keyFrameInfo.poseAffinetInv.matrix.val[6];
    camera.rot[6] = keyFrameInfo.poseAffinetInv.matrix.val[8];
    camera.rot[7] = keyFrameInfo.poseAffinetInv.matrix.val[9];
    camera.rot[8] = keyFrameInfo.poseAffinetInv.matrix.val[10];
    // translation
    camera.trans[0] = keyFrameInfo.poseAffinetInv.matrix.val[3];
    camera.trans[1] = keyFrameInfo.poseAffinetInv.matrix.val[7];
    camera.trans[2] = keyFrameInfo.poseAffinetInv.matrix.val[11];

    // write to .CAM file
    output_file
      << camera.trans[0] << " " << camera.trans[1] << " " << camera.trans[2] << " "
      << camera.rot[0] << " " << camera.rot[1] << " " << camera.rot[2] << " "
      << camera.rot[3] << " " << camera.rot[4] << " " << camera.rot[5] << " "
      << camera.rot[6] << " " << camera.rot[7] << " " << camera.rot[8] << std::endl;
    output_file
      << camera.flen << " "
      << camera.dist[0] << " " << camera.dist[1] << " "
      << camera.paspect << " "
      << camera.ppoint[0] << " " << camera.ppoint[1] << std::endl;    
    ```
  - keyFrame00xx_color.png
  - keyFrame00xx_depth.png

A quick guide on how to use these applications can be found on our project [website](http://www.gcc.tu-darmstadt.de/home/proj/texrecon/).

By starting the application without any parameters and you will get a
description of the expected file formats and optional parameters.

run log:

```
./apps/texrecon/texrecon (built on Jun 14 2022, 14:09:57)
Load and prepare mesh: 
PLY Loader: comment Export generated by faceRecon
Reading PLY: 15122 verts... 29646 faces... done.
Generating texture views: 
        Loading 100%... done. (Took 0.4s)                                       
Building adjacency graph: 
        Adding edges 100%... done. (Took 0.027s)                                
	44171 total edges.
View selection:
	Building BVH from 29646 faces... done. (Took: 17 ms)
        Calculating face qualities 100%... done. (Took 1.32s)                   
        Postprocessing face infos 100%... done. (Took 0.028s)                   
	Maximum quality of a face within an image: 2.93522
	Clamping qualities to 0.906486 within normalization.
	Writing data cost file... done.
	Optimizing:
		Time[s]	Energy
		0	28866
		0	27290
		0	26682
		0	26370
		0	26199
		0	26132
		0	26063
		0	26031
		0	26003
		0	25980
		0	25964
		0	25952
		0	25944
		0	25931
		0	25923
	399 faces have not been seen
	Took: 2.031s
Generating texture patches:
	Running... done. (Took 0.078s)
	308 texture patches.
Running global seam leveling:
	Create matrices for optimization...  done.
	Lhs dimensionality: 18335 x 18335
	Calculating adjustments:
		Color channel 0: CG took 250 iterations. Residual is 9.7671e-05
		Color channel 1: CG took 245 iterations. Residual is 9.55677e-05
		Color channel 2: CG took 223 iterations. Residual is 9.25516e-05
		Took 0.054 seconds
        Adjusting texture patches 100%... done. (Took 0.014s)                   
Running local seam leveling:
        Blending texture patches 100%... done. (Took 0.068s)                    
Generating texture atlases:
	Sorting texture patches... done.
	Working on atlas 2 100%... done.
	Finalizing texture atlases... done. (Took: 0.053s)
Building objmodel:
	Saving model... done.
Whole texturing procedure took: 2.935s
```

Troubleshooting
--------------------------------------------------------------------------------

When you encounter errors or unexpected behavior please make sure to switch
the build type to debug e.g. `cmake -DCMAKE_BUILD_TYPE=DEBUG ..`, recompile
and rerun the application. Because of the computational complexity the default
build type is RELWITHDEBINFO which enables optimization but also ignores
assertions. However, these assertions could give valuable insight in failure cases.


License, Patents and Citing
--------------------------------------------------------------------------------
Our software is licensed under the BSD 3-Clause license, for more details see
the LICENSE.txt file.

If you use our texturing code for research purposes, please cite our paper:
```
@inproceedings{Waechter2014Texturing,
  title    = {Let There Be Color! --- {L}arge-Scale Texturing of {3D} Reconstructions},
  author   = {Waechter, Michael and Moehrle, Nils and Goesele, Michael},
  booktitle= {Proceedings of the European Conference on Computer Vision},
  year     = {2014},
  publisher= {Springer},
}
```

Contact
--------------------------------------------------------------------------------
If you have trouble compiling or using this software, if you found a bug or if
you have an important feature request, please use the issue tracker of github:
https://github.com/nmoehrle/mvs-texturing

For further questions you may contact us at
mvs-texturing(at)gris.informatik.tu-darmstadt.de
