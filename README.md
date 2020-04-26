# Swift_AR_Dicee


### AR 종류  - 먼저 swift project를 만들 때 AR을 선택한다


- SceneKit (3D)
- SpriteKit (2D)
- Metal(Computer) - GPU

# SceneKit (3D)

### Apple이 제공하는 Skeleton Code

```swift
//
//  ViewController.swift
//  ARDicee
//
//  Created by shin seunghyun on 2020/04/26.
//  Copyright © 2020 shin seunghyun. All rights reserved.
//

import UIKit
import SceneKit
import ARKit

class ViewController: UIViewController, ARSCNViewDelegate {

    @IBOutlet var sceneView: ARSCNView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Debugging 현재 찾고 있는 부분을 보여줌
                sceneView.debugOptions = [ARSCNDebugOptions.showFeaturePoints]
        
        // Set the view's delegate
        sceneView.delegate = self
        
        // Create a new scene
        let scene: SCNScene = SCNScene(named: "art.scnassets/ship.scn")!
        
        // Set the scene to the view
        sceneView.scene = scene
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        // Create a session configuration
        let configuration: ARWorldTrackingConfiguration = ARWorldTrackingConfiguration()
                //어떤식으로 변경될지 정하기
        configuration.planeDetection = .horizontal
        
        /** Configuration을 통해서 화면이 어떻게 출력될지 정할 수 있다. ***/
        print("ARConfiguration is supported = \(ARConfiguration.isSupported)")
        print("World Tracking is supported = \(ARWorldTrackingConfiguration.isSupported)")
    
        
        // Run the view's session
        sceneView.session.run(configuration)
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        
        // Pause the view's session
        sceneView.session.pause()
    }

}
```

- ARConfiguration은 기본적으로 어떤 식으로 AR이 표현될지 정해준다.

### Cube 그리기

```swift
/** Create AR Object **/
let cube: SCNBox = SCNBox(width: 0.1, height: 0.1, length: 0.1, chamferRadius: 0.01)

/**Object의 design, color **/
let material: SCNMaterial = SCNMaterial()
material.diffuse.contents = UIColor.red
cube.materials = [material]

/** 3D space의 위치 **/
let node: SCNNode = SCNNode()
//X, Y, Z
node.position = SCNVector3(0, 0.1, -0.5)
//geometry
node.geometry = cube

/** 실제로 보여줌 **/
/** root node - child node  **/
sceneView.scene.rootNode.addChildNode(node)
sceneView.autoenablesDefaultLighting = true  //Object가 3D처럼 보일려면 shadow나 shade 값을 줘야한다.
```

### 달 그리기

- 먼저 art.scnassets 에 가서 jpg 이미지를 넣는다.

```swift
// Set the view's delegate
sceneView.delegate = self

/** Create AR Object **/
let sphere: SCNSphere = SCNSphere(radius: 0.2)

/**Object의 design, color **/
let material: SCNMaterial = SCNMaterial()
material.diffuse.contents = UIColor.red
material.diffuse.contents = UIImage(named: "art.scnassets/moon.jpg")
sphere.materials = [material]

/** 3D space의 위치 **/
let node: SCNNode = SCNNode()
//X, Y, Z
node.position = SCNVector3(0, 0.1, -0.5)
//geometry
node.geometry = sphere

/** 실제로 보여줌 **/
/** root node - child node  **/
sceneView.scene.rootNode.addChildNode(node)
sceneView.autoenablesDefaultLighting = true  //Object가 3D처럼 보일려면 shadow나 shade 값을 줘야한다.

// Create a new scene
//let scene: SCNScene = SCNScene(named: "art.scnassets/ship.scn")!

// Set the scene to the view
//sceneView.scene = scene
```

### Free Resource로 3D Object 만드는 법

I. 무료 사이트 

[https://www.turbosquid.com/](https://www.turbosquid.com/)

⇒ 사람들이 만든 무료 3D Modeling

⇒ `.dae` format을 찾는다 

⇒ `collada` 라고 불리우기도 한다 

II. 무료사이트에서 받은 파일을 xCode에 가지고 온다.

⇒ Editor → Convert to SceneKit file format 을 클릭해서 코딩이 가능한 object로 변경해준다. 확장자명 `scn` 인 파일을 만들어야함. 


### Dice App

- Complete Code Before Refactoring

```swift
//
//  ViewController.swift
//  ARDicee
//
//  Created by shin seunghyun on 2020/04/26.
//  Copyright © 2020 shin seunghyun. All rights reserved.
//

import UIKit
import SceneKit
import ARKit

class ViewController: UIViewController, ARSCNViewDelegate {
    
    var diceArray: [SCNNode] = [SCNNode]()
    
    @IBOutlet var sceneView: ARSCNView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        sceneView.debugOptions = [ARSCNDebugOptions.showFeaturePoints]
        
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        /** Configuration을 통해서 화면이 어떻게 출력될지 정할 수 있다. ***/
        print("ARConfiguration is supported = \(ARConfiguration.isSupported)")
        print("World Tracking is supported = \(ARWorldTrackingConfiguration.isSupported)")
        
        // Create a session configuration
        let configuration: ARWorldTrackingConfiguration = ARWorldTrackingConfiguration()
        
        //어떤식으로 변경될지 정하기
        configuration.planeDetection = .horizontal
        
        // Run the view's session
        sceneView.session.run(configuration)
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        
        // Pause the view's session
        sceneView.session.pause()
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesBegan(touches, with: event)
        
        if let touch: UITouch = touches.first {
            let touchLocation = touch.location(in: sceneView) //sceneView 안에만 detect하게 걸어둠
            let results = sceneView.hitTest(touchLocation, types: .existingPlaneUsingExtent)
            
            if !results.isEmpty {
                print("touched the plane")
            } else {
                print("touched somewhere else")
            }
            
            //Click할 때마다 dice 생겨나게하기
            if let hitResult: ARHitTestResult = results.first {
                
                print(hitResult) //온갖 좌표값이 출력된다.
                
                sceneView.delegate = self
                sceneView.autoenablesDefaultLighting = true  //Object가 3D처럼 보일려면 shadow나 shade 값을 줘야한다.
                let diceScene: SCNScene = SCNScene(named: "art.scnassets/diceCollada.scn")!
                let diceNode: SCNNode? = diceScene.rootNode.childNode(withName: "Dice", recursively: true)
                diceNode?.position = SCNVector3(
                    hitResult.worldTransform.columns.3.x, //x
                    hitResult.worldTransform.columns.3.y + (diceNode?.boundingSphere.radius)!, //y
                    hitResult.worldTransform.columns.3.z //z
                )
                
                diceArray.append(diceNode!)
                
                sceneView.scene.rootNode.addChildNode(diceNode!)
                
                roll(dice: diceNode!)

                
            }
            
        }
        
    }
    
    func rollAll() {
        
        if !diceArray.isEmpty {
            for dice in diceArray {
                roll(dice: dice)
            }
        }
        
    }
    
    //Spinning
    func roll(dice: SCNNode) {
        let randomX = Float(arc4random_uniform(4) + 1) * (Float.pi / 2)
        let randomZ = Float(arc4random_uniform(4) + 1) * (Float.pi / 2)
        
        dice.runAction(
            SCNAction.rotateBy(
                x: CGFloat(randomX * 5),
                y: 0,
                z: CGFloat(randomZ * 5),
                duration: 0.5
            )
        )
    }
    
    @IBAction func rollAgain(_ sender: UIBarButtonItem) {
        rollAll()
    }
    
    @IBAction func removeAllDice(_ sender: UIBarButtonItem) {
        
        if !diceArray.isEmpty {
            for dice in diceArray {
                dice.removeFromParentNode()
            }
        }
        
    }
    
    override func motionEnded(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        super.motionEnded(motion, with: event)
        rollAll()
    }
    
    //특정 Anchor를 찾았을 때를 알려줌, 전체 sceneView의 scene을 의미한다.
    //클릭할 위치를 알려주기 위해 만듬.
    /**
     
     ARAnchor : A position and orientation of something of interest in the physical environment.
     ARPlaneAnchor : A 2D surface that ARKit detects in the physical environment.
     
     **/
    func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor) {
        if anchor is ARPlaneAnchor {
            
            /***
             - 아래 전체 logic 설명
             - ARAnchor라는 좌표에서 2DPlaneAnchor를 찾음
             - gridMaterial을 만들어주고 node를 추가시켜준다.
             ***/
            
            /** SCN Plane **/
            let planeAnchor: ARPlaneAnchor = anchor as! ARPlaneAnchor
            //❗️X,Y 가 아니라 X,Z다.
            let plane: SCNPlane = SCNPlane(width: CGFloat(planeAnchor.extent.x), height: CGFloat(planeAnchor.extent.z))
            
            /** node initialization **/
            let planeNode: SCNNode = SCNNode()
            //위치 설정
            planeNode.position = SCNVector3(x: planeAnchor.center.x, y: 0, z: planeAnchor.center.z)
            planeNode.transform = SCNMatrix4MakeRotation( -Float.pi / 2, 1, 0, 0 )
            
            /** design **/
            let gridMaterial: SCNMaterial = SCNMaterial()
            gridMaterial.diffuse.contents = UIImage(named: "art.scnassets/grid.scn")
            planeNode.geometry = plane
            
            /** add the created node **/
            node.addChildNode(planeNode)
            
        } else {
            return
        }
    }
    
}
```

- Complete Project After Refactoring

```swift
//
//  ViewController.swift
//  ARDicee
//
//  Created by shin seunghyun on 2020/04/26.
//  Copyright © 2020 shin seunghyun. All rights reserved.
//

import UIKit
import SceneKit
import ARKit

class ViewController: UIViewController, ARSCNViewDelegate {
    
    var diceArray: [SCNNode] = [SCNNode]()
    
    @IBOutlet var sceneView: ARSCNView!
    
    //MARK: - LifeCycleMethod
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        sceneView.debugOptions = [ARSCNDebugOptions.showFeaturePoints]
        
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        /** Configuration을 통해서 화면이 어떻게 출력될지 정할 수 있다. ***/
        print("ARConfiguration is supported = \(ARConfiguration.isSupported)")
        print("World Tracking is supported = \(ARWorldTrackingConfiguration.isSupported)")
        
        // Create a session configuration
        let configuration: ARWorldTrackingConfiguration = ARWorldTrackingConfiguration()
        
        //어떤식으로 변경될지 정하기
        configuration.planeDetection = .horizontal
        
        sceneView.delegate = self
        sceneView.autoenablesDefaultLighting = true  //Object가 3D처럼 보일려면 shadow나 shade 값을 줘야한다.
        sceneView.session.run(configuration)
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        
        // Pause the view's session
        sceneView.session.pause()
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesBegan(touches, with: event)
        
        if let touch: UITouch = touches.first {
            let touchLocation = touch.location(in: sceneView) //sceneView 안에만 detect하게 걸어둠
            let results = sceneView.hitTest(touchLocation, types: .existingPlaneUsingExtent)
            
            if !results.isEmpty {
                print("touched the plane")
            } else {
                print("touched somewhere else")
            }
            
            //Click할 때마다 dice 생겨나게하기
            if let hitResult: ARHitTestResult = results.first {
                addDice(atLocation: hitResult)
            }
            
        }
        
    }
    
    func addDice(atLocation location: ARHitTestResult){
        let diceScene: SCNScene = SCNScene(named: "art.scnassets/diceCollada.scn")!
        let diceNode: SCNNode? = diceScene.rootNode.childNode(withName: "Dice", recursively: true)
        diceNode?.position = SCNVector3(
            location.worldTransform.columns.3.x, //x
            location.worldTransform.columns.3.y + (diceNode?.boundingSphere.radius)!, //y
            location.worldTransform.columns.3.z //z
        )
        diceArray.append(diceNode!)
        sceneView.scene.rootNode.addChildNode(diceNode!)
        roll(dice: diceNode!)
    }
    
    func rollAll() {
        if !diceArray.isEmpty {
            for dice in diceArray {
                roll(dice: dice)
            }
        }
    }
    
    //Spinning
    func roll(dice: SCNNode) {
        let randomX = Float(arc4random_uniform(4) + 1) * (Float.pi / 2)
        let randomZ = Float(arc4random_uniform(4) + 1) * (Float.pi / 2)
        
        dice.runAction(
            SCNAction.rotateBy(
                x: CGFloat(randomX * 5),
                y: 0,
                z: CGFloat(randomZ * 5),
                duration: 0.5
            )
        )
    }
    
    @IBAction func rollAgain(_ sender: UIBarButtonItem) {
        rollAll()
    }
    
    @IBAction func removeAllDice(_ sender: UIBarButtonItem) {
        if !diceArray.isEmpty {
            for dice in diceArray {
                dice.removeFromParentNode()
            }
        }
    }
    
    override func motionEnded(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        super.motionEnded(motion, with: event)
        rollAll()
    }
    
    
    //MARK: - ARSCNViewDelegateMethod
    
    //특정 Anchor를 찾았을 때를 알려줌, 전체 sceneView의 scene을 의미한다.
    //클릭할 위치를 알려주기 위해 만듬.
    /**
     
     ARAnchor : A position and orientation of something of interest in the physical environment.
     ARPlaneAnchor : A 2D surface that ARKit detects in the physical environment.
     
     **/
    func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor) {
        /** SCN Plane **/
        
        guard let planeAnchor: ARPlaneAnchor = anchor as? ARPlaneAnchor else { return }
        /***
         - 아래 전체 logic 설명
         - ARAnchor라는 좌표에서 2DPlaneAnchor를 찾음
         - gridMaterial을 만들어주고 node를 추가시켜준다.
         ***/
        
        let planeNode: SCNNode = createPlane(withPlaneAnchor: planeAnchor)
        
        /** add the created node **/
        node.addChildNode(planeNode)
    }
    
    //MARK: - Plane Render Methods
    func createPlane(withPlaneAnchor planeAnchor: ARPlaneAnchor) -> SCNNode {

        //❗️X,Y 가 아니라 X,Z다.
        let plane: SCNPlane = SCNPlane(width: CGFloat(planeAnchor.extent.x), height: CGFloat(planeAnchor.extent.z))
        
        /** node initialization **/
        let planeNode: SCNNode = SCNNode()
        //위치 설정
        planeNode.position = SCNVector3(x: planeAnchor.center.x, y: 0, z: planeAnchor.center.z)
        planeNode.transform = SCNMatrix4MakeRotation( -Float.pi / 2, 1, 0, 0 )
        
        /** design **/
        let gridMaterial: SCNMaterial = SCNMaterial()
        gridMaterial.diffuse.contents = UIImage(named: "art.scnassets/grid.scn")
        planeNode.geometry = plane
        
        return planeNode
    }
    
}
```
