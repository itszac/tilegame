<script>
    import * as BABYLON from "babylonjs";
    import * as BABYLON_GUI from "babylonjs-gui";
    import { onMount } from 'svelte';
    import * as Ammo from "ammo.js";
    import _ from "lodash";
    import centroid from 'polygon-centroid';
    import * as turf from "@turf/helpers";
    import nearestPointOnLine from "@turf/nearest-point-to-line";
    var polygonPolygon = require('intersects/polygon-polygon');
    var pointPolygon = require('intersects/point-polygon');
    var convexHull = require('monotone-convex-hull-2d')
    const geometric = require("geometric");




    let canvas = null
    let engine = null
    let addPolygon = () => {}
    let addPiece = () => {}
    let loadSolution = () => {}
    const materials = [];
    var glitterTexture = null;
    var asphaltTexture = null;
    var speckledTexture = null;
    console.log(BABYLON.GUI)
    var solutions = [
      {
        name:"square",
        positions: [[1,1],[20,20],[100,100],[150,150],[200,200],[250,200]],
        outline: [[0,0],[100,0],[100,100],[0,100],[0,0]]
      },
      {
        name:"triangle",
        positions: [[1,1],[20,20],[100,100],[150,150],[200,200],[250,200]],
        outline: [[0,0],[565.68,0],[565.68,565.68],[0,0]]
      },
      {
        name:"trapezoid",
        positions: [[1,1],[20,20],[100,100],[150,150],[200,200],[250,200]],
        outline: [[0,0],[141.42,0],[141.42,282.82],[0,282.82],[0,0]]
      },
      {
        name:"pentagon",
        positions: [[1,1],[20,20],[100,100],[150,150],[200,200],[250,200]],
        outline: [[0,0],[70.71,0],[141.42,141.42],[300,70.71],[0,0]]
      },
      {
        name:"rectangle",
        positions: [[1,1],[20,20],[100,100],[150,150],[200,200],[250,200]],
        outline: [[0,0],[200,0],[200,100],[0,100],[0,0]]
      },

    ];
    onMount(() => {
        var canvas = document.getElementById("renderCanvas"); // Get the canvas element
        var engine = new BABYLON.Engine(canvas, true); // Generate the BABYLON 3D engine
        // assume it's 100x100
        //
        var piece = {x: 0, y: 0, width: 100, height: 100}
        var pieces = [piece]
        // start with 9 pieces
        var random_choices_x = []
        var random_choices_y = []
        for (var i = 5; i > 0; i--) {
            var new_pieces = []
            var splitPieceIndex = Math.round(Math.random() * (pieces.length - 1))
            var splitPiece = pieces[splitPieceIndex]
            pieces.splice(splitPieceIndex, 1);
            var xSplit = Math.round(Math.random() * (splitPiece.x + splitPiece.width))
            var ySplit = Math.round(Math.random() * (splitPiece.y + splitPiece.height))
            console.log(xSplit, ySplit)
            pieces.push({x: splitPiece.x, y: splitPiece.y, width: xSplit, height: ySplit})
            pieces.push({x: splitPiece.x + splitPiece.width, y: splitPiece.y, width: xSplit - splitPiece.x, height: ySplit - splitPiece.y})
            pieces.push({x: splitPiece.x + splitPiece.width, y: splitPiece.y + splitPiece.height, width: xSplit - splitPiece.x, height: ySplit - splitPiece.y})
            pieces.push({x: splitPiece.x, y: splitPiece.y + splitPiece.height, width: xSplit - splitPiece.x, height: ySplit - splitPiece.y})
        }
        pieces = [
            {x: 0, y: 0, width: 40, height: 60},
            {x: 40, y: 0, width: 60, height: 60},
            {x: 0, y: 50, width: 30, height: 40},
            {x: 40, y: 50, width: 70, height: 40},
        ]
        console.log(pieces)
        // 7 pieces - 1 parallelogram, 2 large triangles, 2 small triangles, 1 medium triangle, 1 square
        var tangramPieces = [
            [[0,0], [0,50], [50,50]], // medium triangle
            [[0,0], [0,100], [50,50]], // large triangle
            [[0,0], [0,50], [50,50], [50,0]], // square
            [[0,0], [0,50], [25,75], [25,25]], // parallelogram
            [[0,0], [0,50], [25,25]], // small triangle
            [[0,0], [0,100], [50,50]], // large triangle
            [[0,0], [0,50], [25,25]], // small triangle
        ]

        /******* Add the create scene function ******/
        var createScene = function () {

            // Create the scene space
            var scene = new BABYLON.Scene(engine);
            var assetsManager = new BABYLON.AssetsManager(scene);
            var textureTask = assetsManager.addTextureTask("glitter task", "/glitter-texture.jpg");
            console.log('doing an assets manager')
    textureTask.onSuccess = function (task) {
        console.log(task.texture, glitterTexture, 'asdfff')
        glitterTexture = task.texture;
        var glitterMat = new BABYLON.StandardMaterial('glitter', scene);
        glitterMat.diffuseTexture = glitterTexture;
        glitterMat.specularTexture = glitterTexture;
        glitterMat.emissiveTexture = glitterTexture;
        glitterMat.ambientTexture = glitterTexture;
        materials.push(glitterMat);
    }
            var asphaltTask = assetsManager.addTextureTask("asphalt task", "/asphalt-texture.jpg");
            console.log('doing an assets manager')
    asphaltTask.onSuccess = function (task) {
        console.log(task.texture, asphaltTexture, 'asdfff')
        asphaltTexture = task.texture;
        var asphaltMat = new BABYLON.StandardMaterial('asphalt', scene);
        asphaltMat.diffuseTexture = asphaltTexture;
        asphaltMat.specularTexture = asphaltTexture;
        asphaltMat.emissiveTexture = asphaltTexture;
        asphaltMat.ambientTexture = asphaltTexture;
        materials.push(asphaltMat);
    }
            var speckledTask = assetsManager.addTextureTask("speckled task", "/speckled-texture-a.jpg");
            console.log('doing an assets manager')
    speckledTask.onSuccess = function (task) {
        console.log(task.texture, speckledTexture, 'asdfff')
        speckledTexture = task.texture;
        var speckledMat = new BABYLON.StandardMaterial('speckled', scene);
        speckledMat.diffuseTexture = speckledTexture;
        speckledMat.specularTexture = speckledTexture;
        speckledMat.emissiveTexture = speckledTexture;
        speckledMat.ambientTexture = speckledTexture;
        materials.push(speckledMat);
    }
    assetsManager.load();
            var blueMat = new BABYLON.StandardMaterial('blue', scene);
            blueMat.diffuseColor = new BABYLON.Color3.FromHexString("#072BB8");
            materials.push(blueMat)
            var gravityVector = new BABYLON.Vector3(0,0, 0);
            //var physicsEngine = scene.enablePhysics(gravityVector, new BABYLON.AmmoJSPlugin(null,Ammo));
            scene.collisionsEnabled = true;

            // Add a camera to the scene and attach it to the canvas


            // Add lights to the scene
            var light = new BABYLON.DirectionalLight("DirectionalLight", new BABYLON.Vector3(0, -1, 0.5), scene);

    var ground = BABYLON.MeshBuilder.CreateBox("ground", {width: 4000, height: 1, depth: 4000}, scene);
    ground.position.y = -5.0;

    var groundMat = new BABYLON.StandardMaterial("groundMat", scene);
    //groundMat.diffuseColor = new BABYLON.Color3(0.5, 0.5, 0.5);
    //groundMat.emissiveColor = new BABYLON.Color3(0.2, 0.2, 0.2);
    groundMat.backFaceCulling = false;
    ground.material = groundMat;
    ground.receiveShadows = true;
    ground.collisionsEnabled = true;



    //ground.physicsImpostor = new BABYLON.PhysicsImpostor(ground, BABYLON.PhysicsImpostor.BoxImpostor, { mass: 0, friction: 10.0, restitution: 0 }, scene);

            return scene;
        };
        /******* End of the create scene function ******/

        var scene = createScene(); //Call the createScene function



var showWorldAxis = (size) => {
    var makeTextPlane = function(text, color, size) {
        var dynamicTexture = new BABYLON.DynamicTexture("DynamicTexture", 50, scene, true);
        dynamicTexture.hasAlpha = true;
        dynamicTexture.drawText(text, 5, 40, "bold 36px Arial", color , "transparent", true);
        var plane = BABYLON.Mesh.CreatePlane("TextPlane", size, scene, true);
        plane.material = new BABYLON.StandardMaterial("TextPlaneMaterial", scene);
        plane.material.backFaceCulling = false;
        plane.material.specularColor = new BABYLON.Color3(0, 0, 0);
        plane.material.diffuseTexture = dynamicTexture;
    return plane;
     };
    var axisX = BABYLON.Mesh.CreateLines("axisX", [ 
      BABYLON.Vector3.Zero(), new BABYLON.Vector3(size, 0, 0), new BABYLON.Vector3(size * 0.95, 0.05 * size, 0), 
      new BABYLON.Vector3(size, 0, 0), new BABYLON.Vector3(size * 0.95, -0.05 * size, 0)
      ], scene);
    axisX.color = new BABYLON.Color3(1, 0, 0);
    var xChar = makeTextPlane("X", "red", size / 10);
    xChar.position = new BABYLON.Vector3(0.9 * size, -0.05 * size, 0);
    var axisY = BABYLON.Mesh.CreateLines("axisY", [
        BABYLON.Vector3.Zero(), new BABYLON.Vector3(0, size, 0), new BABYLON.Vector3( -0.05 * size, size * 0.95, 0), 
        new BABYLON.Vector3(0, size, 0), new BABYLON.Vector3( 0.05 * size, size * 0.95, 0)
        ], scene);
    axisY.color = new BABYLON.Color3(0, 1, 0);
    var yChar = makeTextPlane("Y", "green", size / 10);
    yChar.position = new BABYLON.Vector3(0, 0.9 * size, -0.05 * size);
    var axisZ = BABYLON.Mesh.CreateLines("axisZ", [
        BABYLON.Vector3.Zero(), new BABYLON.Vector3(0, 0, size), new BABYLON.Vector3( 0 , -0.05 * size, size * 0.95),
        new BABYLON.Vector3(0, 0, size), new BABYLON.Vector3( 0, 0.05 * size, size * 0.95)
        ], scene);
    axisZ.color = new BABYLON.Color3(0, 0, 1);
    var zChar = makeTextPlane("Z", "blue", size / 10);
    zChar.position = new BABYLON.Vector3(0, 0.05 * size, 0.9 * size);
};
                    var camera = new BABYLON.ArcRotateCamera("Camera", 0, 0, 2, new BABYLON.Vector3(0,500,50), scene);
            camera.attachControl(canvas, true);
            camera.upperRadiusLimit = 700;
            camera.lowerAlphaLimit = 1;
            camera.upperBetaLimit = 1.5;

        // Register a render loop to repeatedly render the scene
        engine.runRenderLoop(function () {
            scene.render();
        });

        // Watch for browser/canvas resize events
        window.addEventListener("resize", function () {
            engine.resize();
        });
        var x = 1
        var c = 0
        let z = 0
        let movingPiece = null
        let movingPieceLastGoodPosition = null
        let nod = false
        let rejectDrag = false
        let startingPoint = null

        rejectDrag = false
        let dragging = false
        let mP = null
        let slider = null
        let advancedTexture = BABYLON_GUI.AdvancedDynamicTexture.CreateFullscreenUI("UI");
        let panel = new BABYLON_GUI.StackPanel();
        advancedTexture.addControl(panel);
        let mode = null
        let rotateStartingPoint = null
        let solution = null;
        let pieceMeshes = {};
        //movingPieceLastGoodPosition = movingPiece.position.clone()

        const snapToFitTakeOne = (meshTwoDVertices) => {
            //get movingPiceLines
            const meshLines = meshTwoDVertices.map((v, vIndex) => {
                if (vIndex < meshTwoDVertices.length - 1) {
                    return [v, meshTwoDVertices[vIndex + 1]]
                }
            })
            meshLines.push([meshTwoDVertices[meshTwoDVertices.length - 1], meshTwoDVertices[0]]);
            const meshMidPoints = meshLines.map((line) => geometric.lineMidPoint(_.flatten(line)))


            //get all mesh lines besides moving piece
            const candidates = [];
            for (var index = 0; index < scene.meshes.length; index++) {
                var mesh = scene.meshes[index];
                if (mesh === mP || mesh.name === meshName || mesh.name === "ground" || mesh.name === "TextPlane" || mesh.name === "axisX" || mesh.name === "axisY" || mesh.name === "axisZ" || mesh.name === "solution") {
                    continue;
                }
                var twoDVertices = getMeshTwoDVertices(mesh);
                candidates.push(twoDVertices);
            }
            const candidateLines = candidates.map((c) => {
                var lines = c.map((v, vIndex) => {
                    if (vIndex < c.length - 1) {
                        return [v, c[vIndex + 1]]
                    }
                })
                lines.push([c[c.length-1], c[0]]);
                return lines
            });
            //find closest line to each mesh line, might need to do closest point on line
            let closestDistance = Infinity;
            let closestIndex = null;
            let closestLine = null;
            for (var i = candidateLines.length - 1; i >= 0; i--) {
                const candidate = candidateLines[i];
                for (var i = meshMidPoints.length - 1; i >= 0; i--) {
                    const midPoint = meshMidPoints[i];
                    const distance = nearestPointOnLine(turf.point(midPoint), turf.line(candidate));
                    if (distance < closestDistance) {
                        closestDistance = distance;
                        closestLine = candidate;
                        closestIndex = i;
                    }
                }
            }
            const closestMeshLine = meshLines[closestIndex];
            //subdivide piece lines into 10 segments
            // for each segment
        }
        const getSlope = (pointA, pointB) => {
            return ((pointB[1] - pointA[1])/(pointB[0] - pointA[0]));
        }
        const getEdges = (vertices) => {
            const edges = vertices.slice(0,vertices.length -1).map((v, vIndex) => [v, vertices[vIndex + 1]])
            edges.push([vertices[meshTwoDVertices.length - 1], vertices[0]]);
            return edges;
        }

        const snapToFit = (meshTwoDVertices) => {
            // there are two possibilities: 1 is that one of the moving pieces edges has both a similar slope and is close to the edge of a different mesh
            // 2 is that one of the vertices on the moving piece is close to one of the edges of the other moved pieces
            // one hypothosis is that it doesn't matter which we choose from two possibilites, 
            // maybe have a slider that controls the "magneticness"

            //for possibilty 2
            // compare slopes of all edges
            // get distance - will probably have to subdivide line

            // for possibilty one
            // get distance of each vertix to all edges
            // find closest point on closest edge
            // if distance is below cutoff, snap

            // so we need - vertices of moving mesh in 2d world coords, edges of moving mesh in 2d world coords, 
            const meshEdges = getEdges(meshTwoDVertices);
            const meshSlopes = meshEdges.map((edge) => getSlope(edge[0], edge[1]));
            const SLOPE_BUFFER = 0.5;
            const closestEdge = null;
            const closestSlopeDifference = Infinity;
            const closestEdgeDistance = Infinity;
            const DISTANCE_BUFFER = 5
            for (var i = pieces.length - 1; i >= 0; i--) {
                const candidatePiece = pieces[i];
                for (var k = candidatePiece.slopes.length - 1; k >= 0; k--) {
                    const candidateSlope = candidatePiece.slopes[k]
                    for (var j = meshSlopes.length - 1; j >= 0; j--) {
                        const meshSlope = meshSlopes[j];
                        const slopeDifference = meshSlope - candidateSlope;
                        if (slopeDifference < SLOPE_BUFFER && slopeDifference < closestSlopeDifference) {
                            const distance = 1;
                            if (distance < closestEdgeDistance) {
                                closestEdge = i;
                                closestEdgeDistance = distance;
                                closestSlopeDifference = slopeDifference;
                            }
                        }
                    }
                }
            }
            if (closestEdge) {
                //snap piece
                const angle = Math.tan((1+(closestEdgeSlope * closestMeshSlope))/(meshSlope - closestEdgeSlope));
                let delta = closestEdgeDistance;
                // might have to recalculate delta after rotation
                mesh.rotate(angle);
                mesh.position.x = Math.cos(delta)
                mesh.position.z = Math.sin(delta)
            }

        }

        const checkForCollisions = (meshTwoDVertices, meshName) => {

            for (var index = 0; index < scene.meshes.length; index++) {
                var mesh = scene.meshes[index];
                if (mesh === mP || mesh.name === meshName || mesh.name === "ground" || mesh.name === "TextPlane" || mesh.name === "axisX" || mesh.name === "axisY" || mesh.name === "axisZ" || mesh.name === "solution") {
                    continue;
                }
                var twoDVertices = getMeshTwoDVertices(mesh);
                if (polygonPolygon(meshTwoDVertices.map((i)=> _.round(i,2)), twoDVertices.map((i)=> _.round(i,2)))) {
                    console.log('rotate collision!!!', meshTwoDVertices, twoDVertices, mesh.name)
                    return true;
                }
            }
            return false;
        }
        scene.onPointerDown = function (evt, pickResult) {
            dragging = false
        // We try to pick an object
        console.log('pd')
            if (pickResult.hit && pickResult.pickedMesh.name !== "ground") {
                var pickinfo = scene.pick(scene.pointerX, scene.pointerY, function (mesh) { return mesh.name == "ground"; });
                //pickResult.pickedMesh.rotation.y += 2*Math.PI/4;
                var faceId = pickResult.faceId;
                if (faceId > 2) {
                    mode = "rotate";
                } else {
                    mode = "drag";
                }

                console.log('faceid', pickResult.faceId)
                console.log('pappp', scene.pointerX, scene.pointerY)
                mP = pickResult.pickedMesh;
                startingPoint = pickinfo.pickedPoint;
                rotateStartingPoint = scene.pointerX;
                slider = new BABYLON_GUI.Slider();
                slider.left = `${0}px`;;
                slider.top = `${100}px`;
                slider.minimum = -180;
                slider.maximum = 180;
                slider.height = "12px"
                slider.width = "40px";
                //advancedTexture.addControl(slider);
                        setTimeout(function () {
                            camera.detachControl(canvas);
                        }, 0);
            }
        };


        scene.onPointerMove = function (evt) {
            if (mode =="drag") {
                dragging = true
                if (!startingPoint) return
                console.log('pm', evt, scene.pick(scene.pointerX, scene.pointerY))
                if (mP) {
                    var pickinfo = scene.pick(scene.pointerX, scene.pointerY, function (mesh) { return mesh.name == "ground"; });
                    if (pickinfo.hit) {
                        var currentPoint = pickinfo.pickedPoint;
                    var diff = currentPoint.subtract(startingPoint);
                    console.log('dfiff', mP.position, currentPoint, diff)
                    var meshVertices = _.chunk(mP.getVerticesData(BABYLON.VertexBuffer.PositionKind), 3);
                    var seen = {};
                    var meshFilteredVertices = meshVertices.filter((vertex) => {
                        var sV = JSON.stringify([vertex[0], vertex[2]]);
                        var s = !seen[sV];
                        seen[sV] = true;
                        return s
                    })
                    console.log('masdf', meshFilteredVertices)
                    var mbv = meshFilteredVertices.map((v) => BABYLON.Vector3.TransformCoordinates(new BABYLON.Vector3(v[0], v[1], v[2]), mP.getWorldMatrix()))
                    console.log('mbv1', mbv)
                    mbv = mbv.map((v) => {return {x: v.x + diff.x, z: v.z + diff.z}})
                    console.log('mbv', mbv)
                    var meshTwoDVertices = _.flatten(mbv.map((v) => [v.x, v.z]))
                    console.log('compare', meshTwoDVertices, getMeshTwoDVertices(mP, diff))
                    meshTwoDVertices = getMeshTwoDVertices(mP, diff);
                    var allowMove = true
                    var oP = {x: mP.position.x, z: mP.position.z};



                    for (var index = 0; index < scene.meshes.length; index++) {
                        var mesh = scene.meshes[index];
                        var vertices = _.chunk(mesh.getVerticesData(BABYLON.VertexBuffer.PositionKind), 3);
                        var seen = {};
                        var filteredVertices = vertices.filter((vertex) => {
                            var sV = JSON.stringify([vertex[0], vertex[2]]);
                            var s = !seen[sV];
                            seen[sV] = true;
                            return s
                        })
                        var bv = filteredVertices.map((v) => BABYLON.Vector3.TransformCoordinates(new BABYLON.Vector3(v[0], v[1], v[2]), mesh.getWorldMatrix()))
                        var twoDVertices = _.flatten(bv.map((v) => [v.x, v.z]))

                        if (mesh === mP || mesh.name === "ground" || mesh.name === "TextPlane" || mesh.name === "axisX" || mesh.name === "axisY" || mesh.name === "axisZ" || mesh.name === "solution") {
                            continue;
                        }
                        if (polygonPolygon(meshTwoDVertices, twoDVertices)) {
                            console.log('collision!!!', meshTwoDVertices, twoDVertices, mesh.name)
                            allowMove = false
                        }
                

                }}
                if (allowMove) {
                    console.log(oP)
                    mP.position.x = mP.position.x + diff.x
                    mP.position.z = mP.position.z + diff.z
                }
                    }
                startingPoint = currentPoint
        } else if (mode == "rotate") {
            console.log('mode is rotate', scene.pointerX, startingPoint.x, mP.rotation)
            let axis = BABYLON.Axis.Y;
            let oppositeAxis = new BABYLON.Vector3(0,-1,0);
            let multiplier = -1;
            if (scene.pointerX > rotateStartingPoint) {
                axis = new BABYLON.Vector3(0,-1,0);
                oppositeAxis = BABYLON.Axis.Y;
                multiplier = 1;
            }
            //mP.rotate(axis, Math.PI/64, BABYLON.Space.LOCAL);
        var meshVertices = _.chunk(mP.getVerticesData(BABYLON.VertexBuffer.PositionKind), 3);
        var seen = {};
        var meshFilteredVertices = meshVertices.filter((vertex) => {
            var sV = JSON.stringify([vertex[0], vertex[2]]);
            var s = !seen[sV];
            seen[sV] = true;
            return s
        })
        console.log('masdf', meshFilteredVertices)
        var twoDPpoints = meshFilteredVertices.map((vertex) => [vertex[0], vertex[2]])
        var cHull = convexHull(twoDPpoints);
        var convexHullPoints = cHull.map((i) => twoDPpoints[i]);
        console.log('convexHull', cHull, convexHullPoints);
        var mbv = convexHullPoints.map((v) => BABYLON.Vector3.TransformCoordinates(new BABYLON.Vector3(v[0], 10, v[1]), mP.getWorldMatrix()))
        console.log('mbv1', mbv)
        mbv = mbv.map((v) => {return {x: v.x, z: v.z}})
        console.log('mbv', mbv)
        var meshTwoDVertices = mbv.slice(0,3).map((v) => [v.x, v.z])
        const mPPivot = mP.getPivotPoint()
        const worldPivot = BABYLON.Vector3.TransformCoordinates(mPPivot, mP.getWorldMatrix());
        console.log('mppivot', mP.getPivotPoint(), meshTwoDVertices, [worldPivot.x, worldPivot.z])
        meshTwoDVertices = geometric.polygonRotate(meshTwoDVertices, multiplier * 57.2958 * (Math.PI/128), [worldPivot.x, worldPivot.z])
        console.log('rotate result', meshTwoDVertices)
        var allowMove = true
        var oP = {x: mP.position.x, z: mP.position.z};


        allowMove = !checkForCollisions(_.flatten(meshTwoDVertices));

            if (allowMove) {
                mP.rotate(axis, Math.PI/128, BABYLON.Space.LOCAL);
                //testo stuffo

            } else if (checkForCollisions(_.flatten(meshTwoDVertices))) {
                let movecount = 0
                while (checkForCollisions(_.flatten(meshTwoDVertices))) {
                    movecount += 1
                    console.log('move', movecount)
                    meshTwoDVertices = geometric.polygonRotate(meshTwoDVertices, multiplier * -1 * 57.2958 * (Math.PI/128) * (1/100), [worldPivot.x, worldPivot.z])
                }

                mP.rotate(oppositeAxis, movecount * (Math.PI/128) * (1/100), BABYLON.Space.LOCAL);


            }
            rotateStartingPoint = scene.pointerX
        }
        }

        const checkSolved = () => {
            const solutionPolygon = _.flatten(solution.outline);
            for (var index = 0; index < scene.meshes.length; index++) {
                var mesh = scene.meshes[index];
                if (mesh === mP || mesh.name === "ground" || mesh.name === "TextPlane" || mesh.name === "axisX" || mesh.name === "axisY" || mesh.name === "axisZ" || mesh.name === "solution") {
                    continue;
                }
                var vertices = _.chunk(mesh.getVerticesData(BABYLON.VertexBuffer.PositionKind), 3);
                var seen = {};
                var cMFV = vertices.filter((vertex) => {
                    var sV = JSON.stringify([vertex[0], vertex[2]]);
                    var s = !seen[sV];
                    seen[sV] = true;
                    return s
                })
                console.log('masdf', cMFV)
                var cTDP = cMFV.map((vertex) => [vertex[0], vertex[2]])
                var cCHull = convexHull(cTDP);
                var cCHullPoints = cCHull.map((i) => cTDP[i]);
                console.log('convexHull', cCHull, cCHullPoints);
                var cmbv = cCHullPoints.map((v) => BABYLON.Vector3.TransformCoordinates(new BABYLON.Vector3(v[0], 10, v[1]), mesh.getWorldMatrix()))
                console.log('mbv1', cmbv)
                cmbv = cmbv.map((v) => {return {x: v.x, z: v.z}})
                console.log('mbv', cmbv)
                var twoDVertices = cmbv.slice(0,3).map((v) => [v.x, v.z])
                for (var vertexIndex = 0; vertexIndex < twoDVertices.length; vertexIndex++) {
                    const vertex = twoDVertices[vertexIndex];
                    if (!pointPolygon(vertex[0], vertex[1], _.flatten(solutionPolygon))) {
                        console.log('solution rejected', vertex, solutionPolygon)
                        return false;
                    }
                }
            }
            return true;
        }
        scene.onPointerUp = function (evt) {
                        setTimeout(function () {
                            camera.attachControl(canvas);
                        }, 0);
            if (mP) {
                pieceMeshes[mP.name].twoDVertices = getMeshTwoDVertices(mP);
                mP = null;
            }
            mode = null
            if (solution && z >= 7) {
                if (checkSolved()) {
                    console.log('solved!!')
                }
            }

        }

        const addRotateControlToPiece = (piece, centerPoint) => {
            console.log('adding rotate control')
                var mesh = BABYLON.MeshBuilder.CreateDisc('disc', { radius: 10, sideOrientation: BABYLON.Mesh.DOUBLESIDE }, scene);
    var mat = new BABYLON.StandardMaterial('disc-mat', scene);
    var iconName = 'sync-alt'
    var tex = new BABYLON.Texture('https://unpkg.com/@fortawesome/fontawesome-free@5.7.2/svgs/solid/' + iconName + '.svg', scene, false, false);
    tex.hasAlpha = true;
    mat.opacityTexture = tex;
    mat.emissiveColor = BABYLON.Color3.Red();
    mesh.material = mat;
    mesh.position.y = 10.5; //piece y + piece extrusion height + 1
    var pieceCenter = piece.getBoundingInfo().boundingSphere.center;
    mesh.position.x = centerPoint.position.x
    mesh.position.z = centerPoint.position.z
    var axis = new BABYLON.Vector3(1, 0, 0);
var angle = Math.PI/2;
mesh.rotate(axis, angle, BABYLON.Space.LOCAL)
    axis = new BABYLON.Vector3(0, 0, 1);
angle = 54*Math.PI/64;
mesh.rotate(axis, angle, BABYLON.Space.LOCAL)
var newMesh = BABYLON.Mesh.MergeMeshes([piece, mesh, centerPoint], true, false, undefined, false, true);
console.log('axxxx', centerPoint.position)
newMesh.setPivotPoint(new BABYLON.Vector3(centerPoint.position.x, centerPoint.position.y, centerPoint.position.z ))
return newMesh;

        }

        showWorldAxis(100)

        var addCenterPointToMesh = (pieceCorners) => {
            var point = BABYLON.MeshBuilder.CreateSphere("point", {diameter: 5, segments:6}, scene);
            var center = function (arr)
                {
                    var x = arr.map (xy => xy[0]);
                    var y = arr.map (xy => xy[1]);
                    var cx = (Math.min (...x) + Math.max (...x)) / 2;
                    var cy = (Math.min (...y) + Math.max (...y)) / 2;
                    return [cx, cy];
                }
            var c = center(pieceCorners);
            console.log('center', c)
            console.log(pieceCorners.map((p) => ({x: p[0], y: p[1]})))
            c = centroid(pieceCorners.map((p) => ({x: p[0], y: p[1]})))

            point.position.x = c.x;
            point.position.z = c.y;
            point.position.y = 6;
            window.cp = point;
            return point;
        }

        const getMeshTwoDVertices = (mesh, diff) => {
            var vertices = _.chunk(mesh.getVerticesData(BABYLON.VertexBuffer.PositionKind), 3);
            var seen = {};
            var cMFV = vertices.filter((vertex) => {
                var sV = JSON.stringify([vertex[0], vertex[2]]);
                var s = !seen[sV];
                seen[sV] = true;
                return s
            })
            console.log('masdf', cMFV)
            var cTDP = cMFV.map((vertex) => [vertex[0], vertex[2]])
            var cCHull = convexHull(cTDP);
            var cCHullPoints = cCHull.map((i) => cTDP[i]);
            console.log('convexHull', cCHull, cCHullPoints);
            var cmbv = cCHullPoints.map((v) => BABYLON.Vector3.TransformCoordinates(new BABYLON.Vector3(v[0], 10, v[1]), mesh.getWorldMatrix()))
            console.log('mbv1', cmbv)
            if (diff) {
                cmbv = cmbv.map((v) => {return {x: v.x + diff.x, z: v.z + diff.z}});
            } else {
                cmbv = cmbv.map((v) => {return {x: v.x, z: v.z}})
            }
            console.log('mbv', cmbv)
            var twoDVertices = _.flatten(cmbv.slice(0,3).map((v) => [v.x, v.z]));
            return twoDVertices
        }


        addPiece = () => {
            var piece = tangramPieces[z]
            if (!piece) {
                console.log('out of pieces')
                return
            }
            let offset = z * 25;
            if (z % 2 === 0) {
                offset = -1 * offset;
            }
            let corners = tangramPieces[z].map((c) => new BABYLON.Vector3(c[0], c[1])).reverse()
            var poly_tri = new BABYLON.PolygonMeshBuilder(`poly${z}`, corners, scene);
            var polygon = poly_tri.build(null, 9);
            //polygon.physicsImpostor = new BABYLON.PhysicsImpostor(polygon, BABYLON.PhysicsImpostor.MeshImpostor, { mass: 0.0001, friction: 10.0, restitution: 0 }, scene);
            polygon.position.y = 10;
            polygon.material = _.sample(materials);
            var centerPoint = addCenterPointToMesh(tangramPieces[z])
            polygon = addRotateControlToPiece(polygon, centerPoint);
            polygon.position.z = polygon.position.z + offset;
            polygon.position.x = polygon.position.x + offset;
            pieceMeshes[polygon.name] = {name: polygon.name, mesh: polygon, twoDVertices: getMeshTwoDVertices(polygon)};
            var twoDVertices = getMeshTwoDVertices(polygon);
            var mc = 0
            while (checkForCollisions(twoDVertices, polygon.name) && mc < 100) {
                twoDVertices = getMeshTwoDVertices(polygon, mc + 5);
                console.log('pz', polygon.position.z)
                mc += 1
            }
            polygon.position.z = mc + 5;

            z += 1
        }

        var solutionIndex = 0;
        loadSolution = () => {
            solution = solutions[0];

            const outline = [];
            for (var i = solution.positions.length - 1; i >= 0; i--) {
                const piece = tangramPieces[i]
                const solutionPosition = solution.positions[i]
                console.log(piece, solutionPosition, pieces, solution.positions)
                for (var j = piece.length - 1; j >= 1; j--) {
                    console.log(piece[j], solutionPosition)
                    outline.push([
                        [piece[j][0]+solutionPosition[0], piece[j][1]+solutionPosition[1]],
                        [piece[j-1][0]+solutionPosition[0], piece[j-1][1]+solutionPosition[1]],
                    ])
                }

            }
            outline.push(outline[0][0]);
            console.log('lines', outline)

            var corners = outline.map((c) => new BABYLON.Vector3(c[1][0], 0, c[1][1])).reverse()
            console.log('cc', corners);
            corners = solution.outline.map((c) => new BABYLON.Vector3(c[0], 10, c[1]))
            //var poly_tri = new BABYLON.PolygonMeshBuilder(`solution`, corners, scene);
            var line_tri = BABYLON.MeshBuilder.CreateLines("solution", {points: corners}, scene);
            //var polygon = poly_tri.build(null, 9);
            //polygon.material=materials[0]
            line_tri.color = new BABYLON.Color3(.3, .3, .2);
            solutionIndex += 1
        }
    })
</script>

<style>
    h1 {
    }
</style>
<main>
    <h1>tile game</h1>
    <canvas id="renderCanvas" touch-action="none" width=600 height=600></canvas>
    <button on:click="{() => addPiece()}">Add Piece</button>
    <button on:click="{() => loadSolution()}">New Puzzle</button>
</main>