<script>
	var scene;
	var renderer;
	var camera;
	var controls;
	var dom_id='2dplot';

	let mouse_pos;
	let mouse_pos_dom;	
	let mouse_pos_dom_color;

	var object;

	let point_value;

	let mouse_inside_div;

	var tmp_points=null;

	export var _width,_height;
	export var data_array;
	export var format;
	export var _color=0xFF0000;

	let color_list=[
		0xFF0000,
		0xff80ed,
		0x065535,
		0x000000,
		0x133337,
		0xffc0cb,
		0x00ffff,
		0xffd700,
		0xff7373,
		0x0000ff,
		0xbada55
	];

	let color_list_css=[
		'#FF0000',
		'#ff80ed',
		'#065535',
		'#000000',
		'#133337',
		'#ffc0cb',
		'#00ffff',
		'#ffd700',
		'#ff7373',
		'#0000ff',
		'#bada55'		
	];	

	import { onMount } from 'svelte';
	import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

	function init(){
		scene = new THREE.Scene();
		renderer= new THREE.WebGLRenderer();	

		function animate(){
			render();
			requestAnimationFrame(function() {
				animate();
			});			
		}
		init_renderer();
		animate();		
	}

	function init_renderer(){
		let width = document.getElementById(dom_id).offsetWidth;
		let height = document.getElementById(dom_id).offsetHeight;
		renderer.setSize(width, height);
		renderer.setClearColor(0xffffff);
		document.getElementById(dom_id).appendChild(renderer.domElement);	

		let offset=width/10;

		let geometry = new THREE.Geometry();
		geometry.vertices.push(new THREE.Vector3(-width*5, -height/2+offset, 0));
		geometry.vertices.push(new THREE.Vector3(width*5, -height/2+offset, 0));
		let material = new THREE.LineBasicMaterial({color: 0x000000});
		let line = new THREE.Line(geometry, material);
		scene.add(line);

		geometry = new THREE.Geometry();
		geometry.vertices.push(new THREE.Vector3(-width/2+offset, -height*5, 0));
		geometry.vertices.push(new THREE.Vector3(-width/2+offset, height*5, 0));
		material = new THREE.LineBasicMaterial({color: 0x000000});
		line = new THREE.Line(geometry, material);
		scene.add(line);	

		var minX=[];
		var minY=[];
		var maxX=[];
		var maxY=[];

		for (var j=0;j<data_array.length;j++){

			var points_array=data_array[j];

			minX[j] = Math.min(...points_array.map(point => point[0]));
			minY[j] = Math.min(...points_array.map(point => point[1]));
			maxX[j] = Math.max(...points_array.map(point => point[0]));
			maxY[j] = Math.max(...points_array.map(point => point[1]));		

			var points=[];
			
			for (var i=0;i<points_array.length;i++){
				
				var point = new THREE.Vector3(
					points_array[i][0]/(maxX[j]-minX[j])*(width-offset)-width/2+offset,
					points_array[i][1]/(maxY[j]-minY[j])*(height-offset)-height/2+offset,
					0);

				points.push(point);
			}

			
			if (format==='scattered_points'){
				geometry = new THREE.Geometry();
				for (var i=0;i<points.length;i++){
					geometry.vertices.push(points[i]);
				}
				material = new THREE.PointsMaterial( { size: 3, sizeAttenuation: false, color: color_list[j] } );
				object=new THREE.Points( geometry, material );
			}
			if (format==='line'){
				geometry = new THREE.BufferGeometry().setFromPoints(points);
				material = new THREE.LineBasicMaterial({ color: color_list[j] });	
				object = new THREE.Line(geometry, material);
			}
			
			scene.add(object);	
		}

		document.getElementById(dom_id).addEventListener('mousemove', onMouseMove, false);

		document.getElementById(dom_id).parentNode.addEventListener('mousemove', function(evt){
			onMousemove2(evt);
		});	

		var canvas = document.getElementById(dom_id);
		var rect = canvas.getBoundingClientRect();	

		function onMousemove2(evt){
			const box = canvas.getBoundingClientRect();
			// console.log(mouse_inside_div);
			if (evt.clientX >= box.left && evt.clientX <= box.right && evt.clientY >= box.top && evt.clientY <= box.bottom) {
				mouse_inside_div=true;
			} else {
				mouse_inside_div=false;
			  	scene.remove(tmp_points);
			  	mouse_pos_dom=null;
			  	tmp_points=null;				
			}		
		}				

		function onMouseMove(event) {

		  var tmpX=event.clientX - rect.left;
		  var tmpY=rect.height- event.clientY + rect.top;

		  mouse_pos=[event.clientX ,event.clientY];

		  var mouseX = width/2-(width/2-tmpX)/camera.zoom+camera.position.x;
		  var mouseY = height/2-(height/2-tmpY)/camera.zoom+camera.position.y;

		  var tmp_found=false;

		  for (var j=0;j<data_array.length;j++){

		  var value_x=mouseX;
		  var value_y=mouseY;  	

		  var points_array=data_array[j];

		  var distance=[];
		  for (var i=0;i<points_array.length;i++){
		  	var point=points_array[i];
		  	var tmp_point_x=point[0]/(maxX[j]-minX[j])*(width-offset)+offset;
		  	var tmp_point_y=point[1]/(maxY[j]-minY[j])*(height-offset)+offset;
		  	var tmp=Math.sqrt(Math.pow(tmp_point_x-value_x,2)+Math.pow(tmp_point_y-value_y,2));
		  	distance.push(tmp);
		  }
		  mouse_pos_dom=document.getElementById(dom_id);

		  var ind_save;
		  var distance_min=Math.min(...distance);
		  if (distance_min<10) {
		  	tmp_found=true;
		  	var ind=distance.indexOf(distance_min);
		  	if (ind!=ind_save){
		  		scene.remove(tmp_points);
			  	ind_save=ind;
			  	var point_x=points_array[ind][0]/(maxX[j]-minX[j])*(width-offset)-width/2+offset;
			  	var point_y=points_array[ind][1]/(maxY[j]-minY[j])*(height-offset)-height/2+offset;
			  	point_value=[points_array[ind][0],points_array[ind][1]];
				var point = new THREE.Vector3(
					point_x,
					point_y,
					0);	

				var geometry = new THREE.Geometry();
				for (var i=0;i<points.length;i++){
					geometry.vertices.push(point);
				}
				var material = new THREE.PointsMaterial( { size: 5, sizeAttenuation: false, color: 0x0000FF } );
				var object=new THREE.Points( geometry, material );	

				mouse_pos_dom_color=color_list_css[j];
				
				scene.add(object);	
				tmp_points=object;			  		
		  	}
		  } else {
		  	if (distance_min>10 && tmp_found==false){
			  	scene.remove(tmp_points);
			  	mouse_pos_dom=null;	  		
		  	}
		  }

		  }

		  if (mouse_inside_div==false){
		  	scene.remove(tmp_points);
		  	mouse_pos_dom=null;
		  	tmp_points=null;
		  	tmp_found=false;
		  }
		}

		const fontLoader = new THREE.FontLoader();

		fontLoader.load('./helvetiker_regular.typeface.json', (font) => {
		  let textGeometry = new THREE.TextGeometry('X', {
		    font: font,
		    size: 24/600*width,
		    height: 5,
		  });
  		  let textMesh = new THREE.Mesh(textGeometry, new THREE.MeshBasicMaterial({ color: 0x000000 }));	
  		  textMesh.position.set(0, -height/2+offset/2, 0);	  
		  scene.add(textMesh);

		  textGeometry = new THREE.TextGeometry('Y', {
		    font: font,
		    size: 24/600*width,
		    height: 5,
		  });
  		  textMesh = new THREE.Mesh(textGeometry, new THREE.MeshBasicMaterial({ color: 0x000000 }));	
  		  textMesh.position.set(-width/2+offset/2, 0, 0);	  
		  scene.add(textMesh);	

		  textGeometry = new THREE.TextGeometry('(0,0)', {
		    font: font,
		    size: 24/600*width,
		    height: 5,
		  });
  		  textMesh = new THREE.Mesh(textGeometry, new THREE.MeshBasicMaterial({ color: 0x000000 }));	
  		  textMesh.position.set(-width/2+offset, -height/2+offset/2, 0);	  
		  scene.add(textMesh);			  	  
		}); 			

		camera = new THREE.OrthographicCamera(width / -2, width / 2, height / 2, height / -2, 1, 1000);	
		camera.position.set(0, 0, 10);		
		camera.lookAt(new THREE.Vector3(0, 0, 0));	

		controls = new AMI.TrackballOrthoControl(camera, document.getElementById(dom_id));
		controls.staticMoving = true;
		controls.noRotate = true;
		camera.controls = controls;					
	}

	function render(){
		controls.update();
		renderer.render(scene,camera);
	}

	onMount(async () => {	
		init();
	});

</script>

<div id="2dplot" style="display: block;width: {_width}px;height: {_height}px;">

{#if mouse_pos_dom!=null && tmp_points!=null}
<div id="point_info" style="display: block; 
							width:50;
							height:50;
							position: absolute;
							z-index: 100;
							text-align: center;
							top: {mouse_pos[1]-30}px;
							left: {mouse_pos[0]}px;
							">
	<p style='font-size:16px;color:{mouse_pos_dom_color};background-color:rgb(228, 240, 182)'>[{point_value[0].toFixed(1)} , {point_value[1].toFixed(1)}]</p>			
</div>
{/if}	

</div>

<style>
	.label {
		color: #FF0000;
		font-family: sans-serif;
		padding: 2px;
		background: rgba( 0, 0, 0, .6 );
	}
</style>