# gradation

```
<html>
<head>
    <title>サイケ模様</title>
    <style type="text/css">*{padding: 0;margin:0;}</style>
</head>
<body>
<canvas id="stage"></canvas>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/105/three.min.js"></script>
<script id="vertexShader" type="x-shader/x-vertex">
        void main() {
            gl_Position = vec4( position, 1.0 );
        }
</script>
<script id="fragmentShader" type="x-shader/x-fragment">
        uniform vec2 u_resolution;
        uniform float u_time;


        // Based on Morgan McGuire @morgan3d
        // https://www.shadertoy.com/view/4dS3Wd
        // noise関数
        float random (in vec2 st) {
            return fract(sin(dot(st.xy,
                                 vec2(12.9898,78.233)))*
                43758.5453123);
        }
        float noise (in vec2 st) {
            vec2 i = floor(st);
            vec2 f = fract(st);

            // Four corners in 2D of a tile
            float a = random(i);
            float b = random(i + vec2(1.0, 0.0));
            float c = random(i + vec2(0.0, 1.0));
            float d = random(i + vec2(1.0, 1.0));

            vec2 u = f * f * (3.0 - 2.0 * f);

            return mix(a, b, u.x) +
                    (c - a)* u.y * (1.0 - u.x) +
                    (d - b) * u.x * u.y;
        }


        // fBm（非整数ブラウン運動）の関数
        #define OCTAVES 6
        float fbm (in vec2 st) {
            // Initial values
            float value = 0.0;
            float amplitude = .5;
            float frequency = 0.;
            //
            // Loop of octaves
            for (int i = 0; i < OCTAVES; i++) {
                value += amplitude * noise(st);
                st *= 2.;
                float num = abs(sin(u_time * 0.1)) - 0.25;
                amplitude *= .5;
                //amplitude *= num;
            }
            return value;
        }

        void main() {
            //vec2 st = gl_FragCoord.xy/u_resolution.xy;

            // 座標の正規化 
            vec2 st = (gl_FragCoord.xy * 2.0 - u_resolution  ) / min(u_resolution.x, u_resolution.y);

            // fBmでノイズ生成
            vec3 fbmcolor = vec3(0.0);
            fbmcolor += fbm(st*2.0);

            // 行列の回転
            float s = sin(u_time * 0.4);           // サインを求める
            float c = cos(u_time * 0.4);           // コサインを求める
            mat2 m = mat2(c, s, -s, c); // 行列に回転用の値をセット
            st *= m;                     // 行列をベクトルに掛け合わせる

            // 着色
            vec3 col = vec3(st.y,st.x,0.2);

            // オーブ
            float l = 0.01 / length(st);
            
            // r,g,b,aで出力
            gl_FragColor = vec4(fbmcolor  + col ,1.0);
        }
</script>
<script>
  let container;
  let camera, scene, renderer;
  let uniforms;

  let init = () => {
    container = document.getElementById('stage');

    camera = new THREE.Camera();
    camera.position.z = 1;

    scene = new THREE.Scene();

    let geometry = new THREE.PlaneBufferGeometry( 2, 2 );

    uniforms = {
      u_time: { type: "f", value: 1.0 },
      u_resolution: { type: "v2", value: new THREE.Vector2() },
      u_mouse: { type: "v2", value: new THREE.Vector2() }
    };

    let material = new THREE.ShaderMaterial( {
      uniforms: uniforms,
      vertexShader: document.getElementById( 'vertexShader' ).textContent,
      fragmentShader: document.getElementById( 'fragmentShader' ).textContent
    } );

    let mesh = new THREE.Mesh( geometry, material );
    scene.add( mesh );

    renderer = new THREE.WebGLRenderer({
      canvas: container,
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
    uniforms.u_resolution.value.x = renderer.domElement.width;
    uniforms.u_resolution.value.y = renderer.domElement.height;

    window.addEventListener("resize", (e) => {
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

  }

  let animate = () => {
    requestAnimationFrame( animate );
    render();
  }

  let render = () => {
    uniforms.u_time.value += 0.05;
    renderer.render( scene, camera );
  }


  init();

  animate();



</script>
</body>
</html>
```
