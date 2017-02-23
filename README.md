# BioProtect-API
API for launching protected applications and folders through BioProtect

BioProtect protects user-specified applications using TouchID or a passcode.

In order for you application/tweak to be compatible with BioProtect, you need to check with BioProtect if an application is protected, and if it is protected, then launch the application via BioProtect launch methods instead of manually calling your own launch methods.

Public API methods
-------------------

    @intreface BioProtectController : NSObject
    +(id)sharedInstance;
    -(BOOL)requiresAuthenticationForIdentifier:(NSString *)bundleIdentifier;
    -(void)launchProtectedApplicationWithIdentifier:(NSString *)bundleIdentifier;
    -(BOOL)requiresAuthenticationForOpeningFolder:(SBFolder *)folder;
    -(void)authenticateForOpeningFolder:(SBFolder *)folder;
    // Advanced
    -(void)authenticateForIdentifier:(NSString *)identifier object:(id)object selector:(SEL)selector arrayOfArgumentsAsNSValuePointers:(NSArray *)arguments;
    @end
    
Launching Applications
----------------------
A typical usage for launching an application would be:

    if ([[obj_getClass("BioProtectController") sharedInstance] requiresAuthenticationForIdentifier:@"com.apple.AppStore"]){
      [[obj_getClass("BioProtectController") sharedInstance] launchProtectedApplicationWithIdentifier:@"com.apple.AppStore"];
    }
    
    
Opening a protected folder:
---------------------------

    if ([[obj_getClass("BioProtectController") sharedInstance] requiresAuthenticationForOpeningFolder:anSBFolderInstance]){
      [[obj_getClass("BioProtectController") sharedInstance] authenticateForOpeningFolder:anSBFolderInstance];
    }

There are 3 requirements that must be fulfilled in order to open a <b>protected folder</b> through BioProtect:

<b>1)</b> <i>SpringBoard MUST be visible.</i><br>
<b>2)</b> <i>Icon list containing the folder MUST be visible.</i><br>
<b>3)</b> <i>The folder should not be already open.</i><br>
  
These should cover most needs. 

Advanced Methods
----------------

If your tweak needs to authenticate for opening an application with specific options/actions etc, then you should use BioProtect API's advanced method:
    
    -(void)authenticateForIdentifier:(NSString *)identifier object:(id)object selector:(SEL)selector arrayOfArgumentsAsNSValuePointers:(NSArray *)arguments;
    
In this case, you authenticate for a protected bundle identifier, passing an object , selector and arguments to be executed after successful authentication. 

For example, if you need to launch a protected application using a specific object and method, e.g. 
    
    FBSOpenApplicationService  -(void)openApplication:(NSString *)bundleIdentifier withOptions:(id)options completion:(id)completionBlock;
    
you would make the BioProtect call like this:
    
    if ([[obj_getClass("BioProtectController") sharedInstance] requiresAuthenticationForIdentifier:@"com.apple.AppStore"]){
        
        FBSOpenApplicationService *appService=[[objc_getClass("FBSOpenApplicationService") alloc] init];
        NSString *bundleIdentifier=@"com.apple.AppStore";
        id options=NULL;
        void (^someBlock)() = ^{
          // do something;
        };
        NSValue *arg1=[NSValue valueWithPointer:&bundleIdentifier];
        NSValue *arg2=[NSValue valueWithPointer:&options];
        NSValue *arg3=[NSValue valueWithPointer:&someBlock];
        NSArray *argumentsArray=[NSArray arrayWithObjects:arg1,arg2,arg3,NULL];
    
        [[obj_getClass("BioProtectController") sharedInstance] authenticateForIdentifier:@"com.apple.AppStore" object:appService selector:@selector(openApplication:withOptions:completion:) arrayOfArgumentsAsNSValuePointers:argumentsArray];
        
    }
		
    
 
