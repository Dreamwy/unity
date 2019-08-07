1. 访问其它物体
1) 使用Find()和FindWithTag()命令
Find和FindWithTag是非常耗费时间的命令，要避免在Update()中和每一帧都被调用的函数中使用。在Start()和Awake()中使用，使用公有变量把它保存下来，以供后面使用。如：
  公有变量= GameObject.Find("物体名"); 或
  公有变量= GameObject.FindWithTag("物体标签名");
2) 把一个物体拖到公有变量上
3) 引用脚本所在的物体的组件上的参数，如下所示：
   transform.position = Vector3(0,5,4);
   renderer.material.color = Color.blue;
   light.intensity = 8;

4) SendMessge()命令：一个调用其它物体上指令(即物体上的脚本中的函数)的方法
5) GetComponent()命令：引用一个组件

实例代码如下所示：

#pragma strict
var thelight:GameObject;
var thetxt:GameObject;
var theCube1:GameObject;
 
function Start () {
	thelight = GameObject.Find("Spotlight");
	thetxt = GameObject.FindWithTag("txt");
	theCube1 = GameObject.Find("Cube1");
}
 
function Update () {
	if(Input.GetKey(KeyCode.L)){
		thelight.light.intensity += 0.01;
		thetxt.GetComponent(GUIText).text = "当前亮度："+thelight.light.intensity;
	}
	
	if(Input.GetKey(KeyCode.K)){
		thelight.light.intensity -= 0.01;
		thetxt.GetComponent(GUIText).text = "当前亮度："+thelight.light.intensity;
	}
	if(Input.GetKey(KeyCode.S)){
		theCube1.SendMessage("OnMouseDown"); //调用theCube1所有脚本中的OnMouseDown函数
	}
}

2. 制作第一人称控制器
    第一人称控制器脚本代码如下所示：

#pragma strict
var speed:float=6.0;
var jumpspeed:float=8.0;
var gravity:float=20.0;
private var movedirection:Vector3=Vector3.zero;
private var grounded:boolean=false;
 
function Start () {
 
}
 
function FixedUpdate () {
	if(grounded){
		movedirection=Vector3(Input.GetAxis("Horizontal"),0,Input.GetAxis("Vertical"));
		
		//Transforms direction from local space to world space.
		movedirection=transform.TransformDirection(movedirection);
		
		movedirection *= speed;
		
		if(Input.GetButton("Jump")){
			movedirection.y = jumpspeed;
		}
	}	
	movedirection.y -= gravity*Time.deltaTime;
	//Move command	
	var controller:CharacterController = GetComponent(CharacterController);
	var flags = controller.Move(movedirection*Time.deltaTime);
	//CollisionFlags.CollidedBelow  底部发生了碰撞
	//CollisionFlags.CollidedSides 四周发生了碰撞
	//CollisionFlags.CollidedAbove 顶端发生了碰撞
	grounded = (flags & CollisionFlags.CollidedBelow)!=0;
}
//强制Unity为本脚本所依附的物体增加角色控制器组件
@script RequireComponent(CharacterController)


3. 导入3DMax模型
    在3DMax(.fbx文件)把显示单位和系统单位设置为cm，再导入到Unity3D(单位为m)中时，则大小相同。

4. 交互功能（自动开关门）
4.1 ControllerColliderHit（碰撞检测）    
    玩家撞到门时，门才能开。

    其相关代码如下所示，需要对模型的动画进行分割，以下脚本绑在玩家身上：

#pragma strict
private var doorisopen:boolean = false;
private var doortimer:float = 0.0;
private var currentdoor:GameObject;
 
var door_open_time:float = 5.0;
var door_open_sound:AudioClip;
var door_shut_sound:AudioClip;
 
function Update(){
	if(doorisopen){
		doortimer += Time.deltaTime;
		
		if(doortimer > door_open_time) {
			doortimer = 0.0;
			door(false,door_shut_sound,"doorshut",currentdoor);
		}
	}
}
 
//检测玩家是否与门相撞
function OnControllerColliderHit(hitt:ControllerColliderHit){ //hitt为与玩家相撞的碰撞体
	print("test1");
	if(hitt.gameObject.tag == "playerDoor" && doorisopen == false){
		print("test2");
		door(true,door_open_sound,"dooropen",hitt.gameObject);
		currentdoor = hitt.gameObject;
	}
}
 
function door(doorcheck:boolean,a_clip:AudioClip,ani_name:String,thisdoor:GameObject){
	doorisopen=doorcheck;
	thisdoor.audio.PlayOneShot(a_clip); //声音取消3D效果，否则无法播放
	thisdoor.transform.parent.animation.Play(ani_name);
}

4.2 RaycastHit(光线投射）
      玩家必须面对门时，光线投射到门时，门才能开。

      其检测碰撞代码如下所示，以下脚本绑在玩家身上：

function Update(){
	var hit:RaycastHit; //hit为与投射光线相遇的碰撞体
	if(Physics.Raycast(transform.position,transform.forward,hit,3)){
  		print("collider happen" + hit.collider.gameObject.tag);
		if(hit.collider.gameObject.tag == "playerDoor"){
			var currentdoor:GameObject = hit.collider.gameObject;
			currentdoor.SendMessage("doorcheck");
			print("call doorcheck");
		}
	}
}


       门身上的脚本如下所示：

#pragma strict
 
private var doorisopen:boolean = false;
private var doortimer:float = 0.0;
//private var currentdoor:GameObject;
 
var door_open_time:float = 5.0;
var door_open_sound:AudioClip;
var door_shut_sound:AudioClip;
 
function Update(){
	if(doorisopen){
		doortimer += Time.deltaTime;
		
		if(doortimer > door_open_time) {
			doortimer = 0.0;
			door(false,door_shut_sound,"doorshut");
		}
	}
}
 
function doorcheck(){
	if(!doorisopen){
		door(true,door_open_sound,"dooropen");
	}
}
 
function door(doorcheck:boolean,a_clip:AudioClip,ani_name:String){
	doorisopen=doorcheck;
	audio.PlayOneShot(a_clip); //声音取消3D效果，否则无法播放
	transform.parent.animation.Play(ani_name);
}


4.3 OnTriggerEnter（触发器）
      只要一进入触发区域，门就可以打开。需要特别注意调整小房子的Box Collider的大小和位置。

      以下脚本绑在门所在的小房子上：

#pragma strict
 
function OnTriggerEnter (col:Collider) { //col为闯入此区域的碰撞体
	if(col.gameObject.tag == "Player"){
		transform.FindChild("door").SendMessage("doorcheck");
	}
}


4.4 OnCollisionEnter
             检测两个普通碰撞体之间的碰撞，需要发力，才能发生碰撞。不能与第一人称控制器发生碰撞。


4.5 交互总结
      1) OnControllerColliderHit

          专门提供给角色控制器使用，以供角色控制器检测与其它物体发生了碰撞。

          它与触发模式的碰撞体不能发生碰撞，而且还会穿过其物体。

           以下脚本绑定在玩家（第一人称控制器）身上。


function OnControllerColliderHit (hit:ControllerColliderHit) {
	if(hit.gameObject.name != "Plane")
	gameObject.Find("wenzi").guiText.text = hit.gameObject.name;
}


      2) RaycastHit
          它不是一个事件，而是一个命令，可以需要的地方调用它来发射一条光线来进行检测。

           可以与触发模式的碰撞体发生碰撞，但也会穿过其物体。   

           以下脚本绑定在玩家（第一人称控制器）身上。            

#pragma strict
var hit:RaycastHit;
function Update () {
	if(Physics.Raycast(transform.position,transform.forward,hit,2)){
		if(hit.collider.gameObject.name != "Plane"){
			gameObject.Find("wenzi").guiText.text = hit.collider.gameObject.name;
		}
	}
}

      3) OnTriggerEnter
          以下脚本绑定在与第一人称控制器相撞的物体上。

#pragma strict
function OnTriggerEnter (col:Collider) {
	gameObject.Find("wenzi").guiText.text = col.gameObject.name;
}

        4） OnCollisionEnter

             检测两个普通碰撞体之间的碰撞，需要发力，才能发生碰撞。不能与第一人称控制器发生碰撞。

5. 播放声音
5.1 audio.Play()
    GameObject有Audio Source组件，且在Audio Source组件中设置了Audio Clip。

     

5.2 audio.PlayOneShot
var door_open_sound:AudioClip; //在检测面板中设置audio clip；当然必须有Audio Source组件,但不需要设置Audio Clip
function myAudioPlay(){
	audio.PlayOneShot(door_open_sound); //声音取消3D效果，否则无法播放
}


5.3 AudioSource.PlayClipAtPoint
      此种播放方式最佳，节约资源。

var collectSound:AudioClip; //在检测面板中设置其值
 
function pickupCell(){
	//临时申请一个AudioSource,播放完了即时销毁
	AudioSource.PlayClipAtPoint(collectSound,transform.position);
}

6. 销毁物体
    Destroy(gameObject);  //即时销毁物体

    Destroy(gameObject,3.0); // 3秒之后销毁物体

7. 给刚体增加速度
     方法一：

#pragma strict
var throwsound:AudioClip;
var coconut_bl:Rigidbody; //为预制物体
 
function Update () {
	if(Input.GetButtonDown("Fire1")){
		audio.PlayOneShot(throwsound);
		var newcoconut:Rigidbody = Instantiate(coconut_bl,transform.position, transform.rotation);
		newcoconut.velocity=transform.forward*30.0; //设置刚体速度
	}
}

8. 忽视碰撞命令
   使用这两个碰撞器碰撞时不发生效果

   Physics.IgnoreCollision(transform.root.collider,newcoconut.collider,true); 



9. 协同程序及动画播放
    协同程序：类似子线程，在运行本代码的同时，运动脚本中的其它代码。

#pragma strict
// this script will be attached to target object
private var beenhit:boolean = false;
private var targetroot:Animation; 
var hitsound:AudioClip;
var resetsound:AudioClip;
var resettime:float=3.0;
 
function Start () {
	targetroot = transform.parent.transform.parent.animation;
}
 
function OnCollisionEnter (theobject:Collision) {
	if(beenhit == false && theobject.gameObject.name == "coconut"){
		StartCoroutine("targethit"); //协同程序：类似子线程，在运行本代码的同时，运动脚本中的其它代码
	}
}
 
function targethit(){
	audio.PlayOneShot(hitsound); // 播放声音
	targetroot.Play("down");  // 播放动画
	beenhit = true;
	
	yield new WaitForSeconds(resettime); // wait for 3s
	
	audio.PlayOneShot(resetsound);
	targetroot.Play("up");
	beenhit=false;
}

10. 动态添加和删除脚本
      

#pragma strict
 
var obj:GameObject;
var myskin:GUISkin;
 
function Start () {
 
}
 
function Update () {
 
}
 
function OnGUI(){
	//GUI.skin = myskin;
	if(GUILayout.Button("Add_Component",GUILayout.Height(40),GUILayout.Width(110))){
		obj.AddComponent("xuanzhuan"); // xuanzhuan为一个脚本xuanzhuan。js
	}
 
	if(GUILayout.Button("Del_Component",GUILayout.Height(40),GUILayout.Width(110))){
		var script:Object = obj.GetComponent("xuanzhuan");
		Destroy(script);
	}		
}

11. 粒子系统
      粒子系统是在三维空间中渲染出来的二维图像，主要用于表现烟、火、水滴和落叶等效果。

      粒子系统由：粒子发射器、粒子动画器和粒子渲染器组成。



12. for循环和数组
       使用Unity3D内建数组：

var cms:Render[];
 
fucntion Update(){
   for(var co:Render in cms){
      co.material.color = Color.red;
   }
}

       使用Unity3D非内建数组Array：
var wenzi_bl:GUIText;
var arr = new Array(9,12,65,58);  //静态数组
var barr = new Array();  //动态数组
 
fucntion Start(){
   barr.Push("push1");
   barr.Push("push2");
   arr.Sort();
   wenzi_bl.text = arr.join(",") + " ,arr长度：" + arr.length+ " >>" + barr + " barr长度:" + barr.length;
}


13. 延时函数及载入新场景
function OnMouseDown () {
	audio.PlayOneShot(beep);
	yield new WaitForSeconds(0.35);
	Application.LoadLevel(levelToload);
}


14. 创建窗口及此窗口中的内容
static var windowSwitch:boolean = false;
private var windowRect = Rect(300,280,240,200);
 
function Update(){
 
	if(Input.GetKeyDown(KeyCode.Escape)){
		windowSwitch = !windowSwitch;
	}
}
 
function OnGUI(){
	if(windowSwitch){
		windowRect = GUI.Window(0,windowRect,WindowContain,"测试窗口");
	}
}
// show window contents
function WindowContain(){
	if(GUI.Button(Rect(70,70,100,20),"关闭音乐")){
		gameObject.Find("Terrain").GetComponent(AudioSource).enabled = false;
	}
	
	if(GUI.Button(Rect(70,100,100,20),"播放音乐")){
		gameObject.Find("Terrain").GetComponent(AudioSource).enabled = true;
	}
	
	if(GUI.Button(Rect(70,130,100,20),"关闭窗口")){
		windowSwitch = false;
	}
	
	if(GUI.Button(Rect(70,160,100,20),"退出游戏")){
		Application.Quit();
	}
	GUI.DragWindow(new Rect(0,0,1000,1000));
 
}

15. 菜单制作方法
      1) GUITexture

      2) GUI对象：GUI.Window和GUISkin。


--------------------- 
版权声明：本文为CSDN博主「Arrow」的原创文章，遵循CC 4.0 by-sa版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/MyArrow/article/details/30213689
