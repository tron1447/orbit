<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>WebXR Plank Experience Game</title>
    <!-- Frameworks for VR rendering and physics/interactions -->
    <script src="https://aframe.io"></script>
    <script src="https://unpkg.com"></script>
  </head>
  <body>
    <a-scene webxr="optionalFeatures: local-floor, bounded-floor">
      
      <!-- Assets -->
      <a-assets>
        <audio id="elevator-click" src="https://google.com" preload="auto"></audio>
      </a-assets>

      <!-- 3D City Environment Background -->
      

      <!-- The Real Ground (Far Below) -->
      <a-plane position="0 -50 0" rotation="-90 0 0" width="200" height="200" color="#111" opacity="0.8"></a-plane>

      <!-- Starting Ground Platform / SkyScraper Base -->
      <a-box position="0 -25 -2" width="10" height="50" depth="10" color="#333333">
        <!-- Elevator Door Frame at the top -->
        <a-box position="0 25.5 2.9" width="3" height="4" depth="0.2" color="#555">
          <!-- Interactive Up Button -->
          <a-box id="up-button" position="1.8 1.5 0.2" width="0.2" height="0.2" depth="0.1" color="#ff0000" 
                 animation__click="property: components.material.material.color; type: color; to: #00ff00; startEvents: click; dur: 200"
                 class="clickable"></a-box>
        </a-box>
      </a-box>

      <!-- THE PLANK (High in the air, sticking out from the skyscraper) -->
      <a-box id="wooden-plank" position="0 0.52 -5" width="0.5" height="0.05" depth="6" color="#8B5A2B" shadow="receive: true; cast: true;"></a-box>
      
      <!-- Warning Sign -->
      <a-text value="DON'T LOOK DOWN!" position="-1 2 -1.8" color="#FFF" width="5" font="mozillavr"></a-text>

      <!-- VR Player Rig with Camera and Controllers -->
      
        <a-camera id="camera" look-controls wasd-controls="acceleration: 20">
          <!-- Visual Cursor for Non-VR testing / Click fallback -->
          <a-cursor raycaster="objects: .clickable" color="#FF0000" position="0 0 -1" scale="0.02 0.02 0.02"></a-cursor>
        </a-camera>
        
        <!-- Controller Inputs for VR headsets -->
        
        
      

    </a-scene>

    <!-- Game Logic Scripts -->
    <script>
      // Simple interactive button logic to trigger VR heights
      const buttonEl = document.querySelector('#up-button');
      const rigEl = document.querySelector('#rig');

      buttonEl.addEventListener('click', function () {
        // Move the player right to the elevator edge facing the plank
        rigEl.setAttribute('animation', {
          property: 'position',
          to: '0 0.5 -1.8',
          dur: 1500,
          easing: 'easeInOutQuad'
        });
        
        // Play audio alert
        const sound = document.querySelector('#elevator-click');
        sound.play().catch(e => console.log("Audio waiting for user gesture"));
      });
    </script>
  </body>
</html>
