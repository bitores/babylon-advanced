# babylon-advanced

无位置：
ArcRotateCamera中有三种独特的属性，分别命名为alpha（弧度），beta（弧度）和半径（一个数字）。 如果你想像一个ArcRotateCamera是一颗绕地球运转的卫星，那么alpha是纵向或横向轴线，β是纬度或上/下轴线，半径是距离地球核心的高度或高度（距离）。

默认情况下，将scene.ambientColor设置为Color3（0,0,0），这意味着没有scene.ambientColor。


	var faceId = pickResult.faceId;
	mesh.getFacetPositionToRef(faceId, ball.position);

	
	var pos  = mesh.getFacetPosition(50); // returns the world position of the mesh 50th facet
    var positions = mesh.getFacetLocalPositions(); // 位置点
    var normals = mesh.getFacetLocalNormals(); // 法线向量

     mesh.getFacetPositionToRef(facetIndex, worldPos);// 获取 指定 nth 三角面的位置
    mesh.getFacetNormalToRef(facetIndex, worldNor);// 获取 指定 nth 三角面的 法线向量
    worldNor.scaleToRef(distance, tmpVector);// 在 法线向量上 缩放的距离
    tmpVector.addToRef(worldPos, boxPos); // 

    var indexes = mesh.getFacetsAtLocalCoordinates(x, y, z);    // returns an array containing the facet indexes
	if (indexes != null) {
	    var worldPos = mesh.getFacetPosition(indexes[0]);       // the world position of the first facet in the block
	}

	var index = mesh.getClosestFacetAtCoordinates(x, y, z); // returns the index of the closest facet to (x, y, z)
	if (index != null) {
	    var worldPos = mesh.getFacetPosition(index);   // the world position of this facet
	}



1、半球光就是环境光，颜色可设置：天光（diffuse），地光(groundColor)，高光(specular)。球形包围光。

2、物体

	2.1 颜色
		无自己的材质时，有光时显示光的颜色。
		有自己的材质时，有光时显示自己的颜色。
		diffuseColor,specularColor

	2.2 贴图
		可贴多个贴图：一是subMesh都可以有自己的贴图;二每个贴图可有 disffuseTexture/specularTexture/AmbientTexture/OpacityTexture/EmissiveTexture

3、
A.intersectsMesh(B, precise) A撞上B, precise: true|false, OBB AABB

A.intersectsPoint(point) A 撞上 点point


4、layerMask

A mesh is rendered only if (mesh.layerMask & this.activeCamera.layerMask) != 0 : the same layer


For lights you can use light.excludedMeshes


yes this is possible:

cam1.layerMask = 2;
cam2.layerMask = 4;
mesh.layerMask = 2
this way mesh will be rendered only on cam1



/// 坐标轴
每个物体都有两个坐标轴：世界坐标轴和局部坐标轴，坐标轴没有位置，只有矢量，所以按轴运动，只是按矢量运动

var displayWorldAxis = function () {
		var origin = new BABYLON.Vector3(0, 0, 0);
        var xNormal = new BABYLON.Vector3(100, 0, 0);
        var yNormal = new BABYLON.Vector3(0, 100, 0);
        var zNormal = new BABYLON.Vector3(0, 0, 100);
        var xAxis = BABYLON.Mesh.CreateLines("xAxis", [origin, xNormal], scene);
        xAxis.color = BABYLON.Color3.Red();
        var yAxis = BABYLON.Mesh.CreateLines("yAxis", [origin, yNormal], scene);
        yAxis.color = BABYLON.Color3.Green();
        var zAxis = BABYLON.Mesh.CreateLines("zAxis", [origin, zNormal], scene);
        zAxis.color = BABYLON.Color3.Blue();
		return null;
	};
	
	var displayMeshAxis = function (mesh) {
		
		mesh.computeWorldMatrix();
        var matrix = mesh.getWorldMatrix();
		var origin = mesh.position;
            	
		// find existing axis for this box and dispose
      	var xAxis = scene.getMeshByName("xAxis"+mesh.name);
        var yAxis = scene.getMeshByName("yAxis"+mesh.name);
        var zAxis = scene.getMeshByName("zAxis"+mesh.name);
        if (xAxis!=null){ xAxis.dispose();}
        if (yAxis!=null){ yAxis.dispose();}
        if (zAxis!=null){ zAxis.dispose();}

		// calculate new normals for this mesh in world coordinate system
		var xNormal=BABYLON.Vector3.TransformCoordinates(new BABYLON.Vector3(100,0,0),matrix);
        var yNormal=BABYLON.Vector3.TransformCoordinates(new BABYLON.Vector3(0,100,0),matrix);
        var zNormal=BABYLON.Vector3.TransformCoordinates(new BABYLON.Vector3(0,0,100),matrix);
		// create axis lines
        xAxis = BABYLON.Mesh.CreateDashedLines("xAxis"+mesh.name, [origin, xNormal],3,10,200, scene, false);
        xAxis.color = BABYLON.Color3.Red();
        yAxis = BABYLON.Mesh.CreateDashedLines("yAxis"+mesh.name, [origin, yNormal],3,10,200, scene, false);
        yAxis.color = BABYLON.Color3.Green();
        zAxis = BABYLON.Mesh.CreateDashedLines("zAxis"+mesh.name, [origin, zNormal],3,10,200, scene, false);
        zAxis.color = BABYLON.Color3.Blue();
		
		scene.render();
		return null;
	};



mesh.freezeNormals() // 设置后，法线方向不变（针对于法线或顶点位置有变时的可能处理，如波浪）


自定义 shader
BABYLON.Effect.ShadersStore["customVertexShader"] = "";
		
BABYLON.Effect.ShadersStore["customFragmentShader"] = ""; 
		
var mat = new BABYLON.ShaderMaterial("ball_shader", scene, "custom", {
    attributes: ["position","normal"]
    ,uniforms: ["worldViewProjection","world","vPositionW","vNormalW","time"]
});



实现两坐标轴中的欧拉角
axis1 = (sphere1.position).subtract(sphere2.position);
axis3 = BABYLON.Vector3.Cross(camera.position, axis1);
axis2 = BABYLON.Vector3.Cross(axis3, axis1);

mesh.scaling.x = axis1.length();
mesh.rotation = BABYLON.Vector3.RotationFromAxis(axis1, axis2, axis3);
mesh.position = ((sphere2.position).add(sphere1.position)).scale(0.5);


<!-- 聚焦 物体：物体居中 -->
this.focus = function ( target, frame ) {

	try{

		var scale = new THREE.Vector3();
		target.matrixWorld.decompose( center, new THREE.Quaternion(), scale );

		if ( frame && target.geometry ) {

			scale = ( scale.x + scale.y + scale.z ) / 3;
			center.add( target.geometry.boundingSphere.center.clone().multiplyScalar( scale ) );
			var radius = target.geometry.boundingSphere.radius * ( scale );
			var pos = object.position.clone().sub( center ).normalize().multiplyScalar( radius * 2 );
			object.position.copy( center ).add( pos );

		}

		object.lookAt( center );

		scope.dispatchEvent( changeEvent );
	} catch(error) {}

};

镜像
var newMesh = mesh.clone();
newMesh.scaling.x *= -1;
newMesh.flipFaces();

、、MirrorTexture