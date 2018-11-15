---
title: iOS数据持久化详解
tags: Object-C
---

### 数据持久化方式：

1、属性列表(plist存储)

2、偏好设置(NSUserDefaults)

3、归档序列化存储

4、沙盒存储

5、Core Data

6、SQLite3

7、FMDB

8、Realm

### 应用场景及使用

 1、属性列表(plist存储)
通常叫做plist文件，用于存储在程序中不经常修改、数据量小的数据，不支持自定义对象存储，支持数据存储的类型为：Array，Dictionary，String，Number，Data，Date，Boolean，通常用来存放接口名、城市名、银行名称、表情名等极少修改的数据

1.1 创建plist文件并添加数据

右击文件目录 ---> 选择"New File..." --->选择"Property List" ---> 输入plist文件名并在窗口中点击Create创建。在创建好的plist文件中选择Root类型并添加测试数据，如图所示:

![创建.png](https://upload-images.jianshu.io/upload_images/1990028-cbaf9156304b9113.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![添加测试数据.png](https://upload-images.jianshu.io/upload_images/1990028-45603fc5c17325ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.2 获取plist文件数据

文件的类型为Array就使用数组获取，类型为Dictionary就使用字典方式获取数据，示例代码：


	/**
 	plist获取数据
 
 	@param sender sender description
 	*/
	- (IBAction)plistGetInfoButtonDidClicked:(UIButton *)sender {
	    //获取plist文件路径
	    NSString *path = [self getPlistFilePath];
	    NSDictionary *dict = [[NSDictionary alloc] initWithContentsOfFile:path];
	    NSLog(@"获取到的plist数据\n%@",dict);
	}

	/**
 	获取plist文件路径

	 @return 路径
 	*/
	- (NSString *)getPlistFilePath {
    	//获取plist文件路径
    	NSString *path = [[NSBundle mainBundle] pathForResource:@"testProperty" ofType:@"plist"];
    	return path;
	}


2 偏好设置(NSUserDefaults)

用于存储用户的偏好设置，同样适合于存储轻量级的用户数据，数据会自动保存在沙盒的Libarary/Preferences目录下，本质上就是一个plist文件，所以同样的不支持自定义对象存储，支持数据存储的类型为：Array，Dictionary，String，Number，Data，Date，Boolean，可以用做检查版本是否更新、是否启动引导页、自动登录、版本号等等，需要注意的是NSUserDefaults是定时的将缓存中的数据写入磁盘，并不是即时写入，为了防止在写完NSUserDefaults后，程序退出导致数据的丢失，可以在写入数据后使用synchronize强制立即将数据写入磁盘

2.1 为NSUserDefaults创建分类

右击文件目录 ---> 选择"New File..." --->选择"Objective-C File" --->选择File Type方式、类并且输入名称

![创建分类2.png](https://upload-images.jianshu.io/upload_images/1990028-cf8da32490580f91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.2 保存、获取数据示例:

	#import <Foundation/Foundation.h>

	NS_ASSUME_NONNULL_BEGIN

	@interface NSUserDefaults (Category)


	/**
	 保存电话号码
	 */
	+ (void)savePhoneNumber:(NSString *)phoneNumber;

	/**
	 获取电话号码

	 @return 电话号码
	 */
	+ (NSString *)getPhoneNumber;
	@end

	NS_ASSUME_NONNULL_END
	
.m文件

	#import "NSUserDefaults+Category.h"

	static NSString * const phoneNumberKey = @"phoneNumberKey";

	@implementation NSUserDefaults (Category)

	#pragma mark - 电话号码
	+ (void)savePhoneNumber:(NSString *)phoneNumber {
    	[[NSUserDefaults standardUserDefaults] setObject:phoneNumber forKey:phoneNumberKey];
    	[[NSUserDefaults standardUserDefaults] synchronize];
	}
	+ (NSString *)getPhoneNumber {
   		return [[NSUserDefaults standardUserDefaults] objectForKey:phoneNumberKey];
	}

3 归档序列化存储

归档可以直接将对象存储为文件，也可将文件直接解归档为对象，相对于plist文件与偏好设置数据的存储更加多样，支持自定义的对象存储，归档后的文件是加密的，也更加的安全，文件存储的位置可以自定义。

3.1 对象支持归档与解归档

创建HJPersonModel类继承自NSObject ，遵守NSCoding或者NSSecureCoding协议，重写两个方法。"supportsSecureCoding"是NSSecureCoding协议下的重要属性，如果采用NSSecureCoding协议，必须重写supportsSecureCoding 方法并返回YES

	#import "HJPersonModel.h"

	@interface HJPersonModel () <NSSecureCoding>

	@end

	@implementation HJPersonModel

	- (void)encodeWithCoder:(nonnull NSCoder *)aCoder {
	    [aCoder encodeObject:_name forKey:@"name"];
	    [aCoder encodeObject:@(_age) forKey:@"age"];
	}

	- (nullable instancetype)initWithCoder:(nonnull NSCoder *)aDecoder {
	    if (self = [super init]) {
	        self.name =  [aDecoder decodeObjectForKey:@"name"];
	        self.age = [[aDecoder decodeObjectForKey:@"age"] integerValue];
	    }
	    return self;
	}

	+ (BOOL)supportsSecureCoding {
	    return YES;
	}

3.2 创建归档解归档工具

创建HJArchiveTool工具类继承自NSObject ，提供两个方法以实现归档解归档

	#import "HJArchiveTool.h"

	@implementation HJArchiveTool

	#pragma mark - 归档解归档

	+ (BOOL)archiveObject:(id)object prefix:(NSString *)prefix {
	    if (!object)
	        return NO;
	    NSError *error;
	    //会调用对象的encodeWithCoder方法
	    NSData *data = [NSKeyedArchiver archivedDataWithRootObject:object
                          requiringSecureCoding:YES
                                          error:&error];
	    if (error)
	        return NO;
    
	    [data writeToFile:[self getPathWithPrefix:prefix] atomically:YES];
	    return YES;
	}

	+ (id)unarchiveClass:(Class)class prefix:(NSString *)prefix {
    
	    NSError *error;
	    NSData *data = [[NSData alloc] initWithContentsOfFile:[self getPathWithPrefix:prefix]];
	    //会调用对象的initWithCoder方法
	    id content = [NSKeyedUnarchiver unarchivedObjectOfClass:class fromData:data error:&error];
	    if (error) {
	        return nil;
	    }
	    return content;
	}


	/**
	 存放的文件路径

	 @return return value description
	 */
	+ (NSString *)getPathWithPrefix:(NSString *)prefix {
    
	    NSString *documentPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
	    NSString *filePathFolder = [documentPath stringsByAppendingPaths:@[@"archiveTemp"]].firstObject;
	    if (![[NSFileManager defaultManager] fileExistsAtPath:filePathFolder]) {
	        [[NSFileManager defaultManager] createDirectoryAtPath:filePathFolder withIntermediateDirectories:YES attributes:nil error:nil];
	    }
	    NSString *path = [NSString stringWithFormat:@"%@/%@.archive",filePathFolder,prefix];
	    return path;
	}


3.3 归档与解归档对象

HJPersonModel类对应的数据归档/解归档操作

	#pragma mark - 归档解归档
	/**
	 归档数据
 
	 @param sender sender description
	 */
	- (IBAction)archiveButtonDidClicked:(UIButton *)sender {
	    HJPersonModel *model = [HJPersonModel new];
	    model.name = @"小李子";
	    model.age = 25;
	    if ([HJArchiveTool archiveObject:model prefix:NSStringFromClass(model.class)]) {
	        NSLog(@"归档成功");
	    } else {
	        NSLog(@"归档失败");
	    }
	}
	/**
	 解归档数据
 
	 @param sender sender description
 	*/
	- (IBAction)unarchiveButtonDidClicked:(UIButton *)sender {
    	id content = [HJArchiveTool unarchiveClass:HJPersonModel.class prefix:NSStringFromClass(HJPersonModel.class)];
    	NSLog(@"%@",content);
	}

4 沙盒存储

可以提高程序的体验度，为用户节约数据流量，主要在用户阅读书籍、听音乐、看视频等，在沙盒中做数据的存储，主要包含文件夹：
Documents: 最常用的目录，存放重要的数据，iTunes同步时会备份该目录
Library/Caches: 一般存放体积大，不重要的数据，iTunes同步时不会备份该目录
Library/Preferences: 存放用户的偏好设置，iTunes同步时会备份该目录
tmp: 用于存放临时文件，在程序未运行时可能会删除该文件夹中的数据，iTunes同步时不会备份该目录

存放音乐文件示例：

4.1 创建HJSandBoxOperationTool工具类继承自NSObject，提供"保存音乐到沙盒路径"和"获取沙盒路径下的音乐"两个方法

	#import "HJSandBoxOperationTool.h"


	@implementation HJSandBoxOperationTool


	/**
	 保存音乐到沙盒路径

	 @param musicUrlStr url地址
	 */
	+ (void)toolToSaveMusicToCache:(NSString *)musicUrlStr withMusicName:(NSString *)name {
    	__weak typeof(self) weakSelf = self;
    	dispatch_async(dispatch_queue_create("musicQueue", DISPATCH_QUEUE_PRIORITY_DEFAULT), ^{
 	       //已经存了
 	       if ([weakSelf alreadySaveMusic:name])
	            return ;
	        NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:musicUrlStr]];
	        [data writeToFile:[weakSelf getPathWithMusicName:name] atomically:YES];
	    });
	}

	/**
	 获取沙盒路径下的音乐
 
	 @param musicName 音乐名称
	 @return 路径
	 */
	+ (NSString *)toolToGetMusicFromCache:(NSString *)musicName {
	    NSString *musicPath = [self getPathWithMusicName:musicName];
	    if ([[NSFileManager defaultManager] fileExistsAtPath:musicPath]) {
	        return musicPath;
	    }
	    return nil;
	}

	/**
	 获取沙盒路径下的音乐
 
	 @return return value description
	 */
	+ (NSString *)getPathWithMusicName:(NSString *)name {
    
	    NSString *documentPath = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) firstObject];
	    NSString *filePathFolder = [documentPath stringsByAppendingPaths:@[@"music"]].firstObject;
	    if (![[NSFileManager defaultManager] fileExistsAtPath:filePathFolder]) {
	        [[NSFileManager defaultManager] createDirectoryAtPath:filePathFolder withIntermediateDirectories:YES attributes:nil error:nil];
	    }
	    NSString *path = [NSString stringWithFormat:@"%@/%@.mp3",filePathFolder,name];
	    return path;
	}


	/**
 	判断是否已经存储了音乐

	 @param musicName 音乐名称
	 @return return value description
	 */
	+ (BOOL)alreadySaveMusic:(NSString *)musicName {
	    NSString *musicPath = [self getPathWithMusicName:musicName];
	    if (![[NSFileManager defaultManager] fileExistsAtPath:musicPath]) {
	        return NO;
	    } else {
	        NSData *data = [NSData dataWithContentsOfFile:musicPath];
	        if (data.length > 0) {
 	           return YES;
 	       }
 	       return NO;
 	   }
	}

4.2 工具类方法调用

	#pragma mark - 沙盒存储

	/**
 	沙盒存数据

	 @param sender sender description
	 */
	- (IBAction)sandBoxSaveButtonDidClicked:(UIButton *)sender {
	    //musicUrl 为.mp3格式的音乐url地址
	    [HJSandBoxOperationTool toolToSaveMusicToCache:musicUrl withMusicName:@"耳朵"];
	}

	/**
	 沙盒取数据

 	@param sender sender description
	 */
	- (IBAction)sendBoxGetInfoButtonDidClicked:(UIButton *)sender {
	    NSString *str = [HJSandBoxOperationTool toolToGetMusicFromCache:@"耳朵"];
	    NSLog(@"%@",str);
	}

5 Core Data

Core Data是框架，并不是数据库，该框架提供了对象关系的映射功能，使得能够将OC对象转换成数据，将数据库中的数据还原成OC对象，在转换的过程中不需要编写任何的SQL语句，在Core Data中有三个重要的概念：

NSPersistentStoreCoordinator：持久化存储协调器，在NSPersistentStoreCoordinator中包含了持久化存储区，在持久化存储区中包含了数据表中的很多数据，持久化存储区的设置通常选择NSSQLiteStoreType，也就是选择SQLite数据库

NSManagedObjectModel：托管对象模型，用于描述数据结构的模型

NSManagedObjectContext：托管对象上下文，在上下文中包含多个托管对象，用于管理对象的生命周期，为何会出现NSManagedObjectContext，原因在于将对象转换成数据是保存到磁盘上的，磁盘与RAM之间传输数据是会有开销，并且磁盘读写的速度没有RAM快，通过托管对象上下文可以迅速的获取到数据，修改了数据之后，需要调用上下文的save方法，将数据写入到数据库中，也就是磁盘中

5.1  Core Data 文件的简单创建

右击文件夹 ---> 选择 "New File..." ---> 选择"Data Model" ---> 输入文件名称并创建。创建后缀名为.xcdatamodeld的模型文件后，添加实体，并在实体中添加属性及类别

![创建Core Data文件.png](https://upload-images.jianshu.io/upload_images/1990028-497b3ca0b155589b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击"Editor"菜单项 ---> 选择"Create NSManagedObject Subclass..."项--->选择创建的Data Model--->选择创建的实体类--->选择需要放置的文件目录--->点击"Create"创建托管对象类。

![构建实体类.png](https://upload-images.jianshu.io/upload_images/1990028-64202a0d1af5b80e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


5.2 单利实例化CoreDataManager

创建HJCoreDataManager继承自NSObject，针对Core Data的基本操作进行封装。.h文件提供单利创建与删除数据库两个方法

	#import <Foundation/Foundation.h>
	#import <CoreData/CoreData.h>


	NS_ASSUME_NONNULL_BEGIN

	@interface HJCoreDataManager : NSObject


	/**
	 单利创建

	 @return HJCoreDataManager
	 */
	+ (instancetype)sharedInstanceManager;

	/**
	 删除数据库
	 */
	+ (void)managerForDelete;

	//......

	@end

	NS_ASSUME_NONNULL_END
	
.m文件为方法的实现

	#import "HJCoreDataManager.h"

	#define HJStrIsEmpty(str) ((str == nil) || ([str isEqualToString: @""]) || (str == NULL) || ([str isKindOfClass:[NSNull class]]))

	@interface HJCoreDataManager ()


	/**
	 数据模型
	 */
	@property (nonatomic, strong) NSManagedObjectModel *objectModel;


	/**
	 持久化数据
	 */
	@property (nonatomic, strong) NSPersistentStoreCoordinator *coordinator;

	/**
	 管理数据的对象
	 */
	@property (nonatomic, strong) NSManagedObjectContext *objectContext;
	@end

	@implementation HJCoreDataManager

	#pragma mark - 单利创建
	+ (instancetype)sharedInstanceManager {
    	static HJCoreDataManager *manager = nil;
    	static dispatch_once_t onceToken;
    	dispatch_once(&onceToken, ^{
        	manager = [[HJCoreDataManager alloc] init];
    	});
    	return manager;
	}

	- (instancetype)init {
	    if (self = [super init]) {
	        //创建托管对象模型
	        NSURL *url = [[NSBundle mainBundle] URLForResource:@"HJCoreData" withExtension:@"momd"];
 	       _objectModel = [[NSManagedObjectModel alloc] initWithContentsOfURL:url];
	        //创建持久化数据协调器
	        _coordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:_objectModel];
	        //关联并创建本地数据库文件
	        [_coordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:[self dbPath] options:nil error:nil];
	        //创建托管对象上下文
	        _objectContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:(NSMainQueueConcurrencyType)];
	        [_objectContext setPersistentStoreCoordinator:_coordinator];
	    }
	    return self;
	}


	/**
	 获取数据库路径

	 @return return value description
	 */
	- (NSURL *)dbPath {
	    NSString *documentPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).lastObject;
	    NSString *dbFolder = [documentPath stringByAppendingPathComponent:@"CoreData"];
	    if (![[NSFileManager defaultManager] fileExistsAtPath:dbFolder]) {
	        [[NSFileManager defaultManager] createDirectoryAtPath:dbFolder withIntermediateDirectories:YES attributes:nil error:nil];
	    }
	    NSURL *dbUrl = [[NSURL fileURLWithPath:dbFolder] URLByAppendingPathComponent:@"db.sqlite"];
	    return dbUrl;
	}

	#pragma mark - 删除数据库
	+ (void)managerForDelete {
 	   NSString *documentPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
	    NSString *filePath = [documentPath stringByAppendingPathComponent:@"CoreData"];
	    if ([[NSFileManager defaultManager] fileExistsAtPath:filePath]) {
	        [[NSFileManager defaultManager] removeItemAtPath:filePath error:nil];
	    }
	}

	//......

	@end

5.3 数据的增删改查

数据增删改查操作的基本封装，.h文件提供方法，.m文件实现方法，需要注意的是，在对数据进行了修改之后都要调用上下文的save方法，将数据写入到数据库中，也就是磁盘中

	//================================ 添加数据 ================================//

	/**
	 获取数据模型

	 @param entityName 数据模型类名称
	 @return return value description
	 */
	+ (NSManagedObject *)getTableWithEntityName:(NSString * _Nonnull)entityName;

	/**
	 保存数据
	 */
	+ (BOOL)save;

	//================================       ================================//

	/**
	 删除数据

	 @param entityName   数据模型类名称
	 @param searchString 属性名的值包含的字符
	 @param attribute    属性名称
	 @return             成功失败
	 */
	+ (BOOL)deleteByEntityName:(NSString * _Nonnull)entityName
               withMaching:(NSString * _Nonnull)searchString
             withAttribute:(NSString * _Nonnull)attribute;

	/**
	更新数据
	
	@param managedObject pojo对象
	@return bool
	*/
	+ (BOOL)updateManagedObject:(NSManagedObject *)managedObject;

	/**
	 查询数据

	 @param entityName   数据模型类名称
	 @param searchString 属性名的值包含的字符
	 @param attribute    属性名称
	 @param sortArribute 按哪个属性名称排序
	 @param ascending    是否升序
	 @return             模型数组
	 */
	+ (NSArray *)selectByEntityName:(NSString * _Nonnull)entityName
                    withMaching:(NSString * _Nullable)searchString
                  withAttribute:(NSString * _Nullable)attribute
                      sortingBy:(NSString * _Nullable)sortArribute
                    isAscending:(BOOL)ascending;
                    
.m文件

	#pragma mark - 获取数据模型
	+ (NSManagedObject *)getTableWithEntityName:(NSString *)entityName {
	    NSManagedObject *managedObject = [NSEntityDescription insertNewObjectForEntityForName:entityName inManagedObjectContext:[HJCoreDataManager sharedInstanceManager].objectContext];
	    return managedObject;
	}

	#pragma mark - 保存数据
	+ (BOOL)save {
	    NSError *error;
	    BOOL success = [[HJCoreDataManager sharedInstanceManager].objectContext save:&error];
 	   return success;
	}

	#pragma mark - 删除数据
	+ (BOOL)deleteByEntityName:(NSString * _Nonnull)entityName
               withMaching:(NSString * _Nonnull)searchString
             withAttribute:(NSString * _Nonnull)attribute {
	    //没有输入删除条件
	    if (HJStrIsEmpty(attribute) || HJStrIsEmpty(searchString)) {
	        return YES;
	    }
	    //查询数据
	    NSArray *array = [self selectByEntityName:entityName
                                  withMaching:searchString
                                withAttribute:attribute
                                    sortingBy:attribute
                                  isAscending:YES];
	    if (array.count > 0) {
	        //删除
	        for (NSManagedObject *object in array) {
 	           [[HJCoreDataManager sharedInstanceManager].objectContext deleteObject:object];
	        }
	    }
 	   //执行保存操作
	    return [HJCoreDataManager save];
	}

	#pragma mark - 更新数据
	+ (BOOL)updateManagedObject:(NSManagedObject *)managedObject {
	    [[HJCoreDataManager sharedInstanceManager].objectContext refreshObject:managedObject mergeChanges:YES];
	    return [HJCoreDataManager save];
	}

	#pragma mark - 查询数据
	+ (NSArray *)selectByEntityName:(NSString * _Nonnull)entityName
                    withMaching:(NSString * _Nullable)searchString
                  withAttribute:(NSString * _Nullable)attribute
                      sortingBy:(NSString * _Nullable)sortArribute
                    isAscending:(BOOL)ascending{
	    NSEntityDescription *entity = [NSEntityDescription entityForName:entityName inManagedObjectContext:[HJCoreDataManager sharedInstanceManager].objectContext];
	    //创建fetch请求
	    NSFetchRequest *fetchRequest = [[NSFetchRequest alloc] init];
	    fetchRequest.entity = entity;
	    //一次性获取完
	    [fetchRequest setFetchBatchSize:0];
	    if (!HJStrIsEmpty(sortArribute)) {
	        //排序
	        NSSortDescriptor *sortDescriptor = [[NSSortDescriptor alloc] initWithKey:sortArribute ascending:ascending selector:nil];
	        NSArray *descriptors = @[sortDescriptor];
	        fetchRequest.sortDescriptors = descriptors;
	    } else {
	        fetchRequest.sortDescriptors = @[];
	    }
    
	    if (!HJStrIsEmpty(searchString) && !HJStrIsEmpty(attribute)) {
	        //某个属性的值包含某个字符串
	        //%K 某个属性的值
       	 //%@ 传递过来的字符串
        	 //模糊查询 contains[cd] 包含某个值 c标识忽略大小写，d标识忽略重音
        	 //查询 ==
	        fetchRequest.predicate = [NSPredicate predicateWithFormat:@"%K == %@",attribute, searchString];
	    }
	    NSError *error;
	    NSFetchedResultsController *fetchedController = [[NSFetchedResultsController alloc] initWithFetchRequest:fetchRequest managedObjectContext:[HJCoreDataManager sharedInstanceManager].objectContext sectionNameKeyPath:nil cacheName:nil];
	    //执行获取操作
	    if ([fetchedController performFetch:&error]) {
	        //获取数据
	        return [fetchedController fetchedObjects];
	    } else {
	        return @[];
	    }
	}



5.4 数据操作例子

封装了之后，调用数据的增删改查就会更加的方便，程序启动后，在"- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions"方法中去调用 "[HJCoreDataManager sharedInstanceManager];"初始化对象，然后根据需求调用所封装的操作数据方法进行读写操作。增删改查示例：

	#pragma mark - Core Data数据操作


	/**
	 增加数据

	 @param sender sender description
	 */
	- (IBAction)coreDataAddInfoButtonDidClicked:(UIButton *)sender {

	    HJCarModel *carModel = (HJCarModel *)[HJCoreDataManager getTableWithEntityName:NSStringFromClass([HJCarModel class])];
    	carModel.carName = @"AE99";
    	carModel.userName = @"一个大孩子";
    	carModel.userID = 11;
    	[HJCoreDataManager save];
	}

	/**
	 删除数据

	 @param sender sender description
	 */
	- (IBAction)coreDataDeleteInfoButtonDidClicked:(UIButton *)sender {
	    BOOL isSuccess = [HJCoreDataManager deleteByEntityName:NSStringFromClass([HJCarModel class])
                                   withMaching:@"AE99"
                                 withAttribute:@"carName"];
	    NSLog(@"%d",isSuccess);
	}


	/**
	 修改数据

	 @param sender sender description
	 */
	- (IBAction)coreDataUpdateInfoButtonDidClicked:(UIButton *)sender {
	    //修改已经保存到数据库中的数据，在修改之前我们应该获取要修改的数据，修改之后对数据进行再次保存
	    NSArray *array = [HJCoreDataManager selectByEntityName:NSStringFromClass([HJCarModel class])
                                     withMaching:nil
                                   withAttribute:nil
                                       sortingBy:@"userID"
                                     isAscending:YES];
		HJCarModel *model = array.firstObject;
		model.userName = @"爱听话的孩子";
		BOOL isSuccess = [HJCoreDataManager updateManagedObject:model];
		NSLog(@"%d",isSuccess);
	}

	/**
	 查询数据

	 @param sender sender description
	 */
	- (IBAction)coreDataSelectInfoButtonDidClicked:(UIButton *)sender {
	    NSArray *array = [HJCoreDataManager selectByEntityName:NSStringFromClass([HJCarModel class])
                                     withMaching:nil
                                   withAttribute:nil
                                       sortingBy:nil
                                     isAscending:YES];
 	   if (array.count > 0) {
	        for (HJCarModel *model in array) {
 	           NSLog(@"%@---%@---%lld", model.userName,model.carName,model.userID);
 	       }
 	   }
    
	}

6 SQLite3

SQLite是轻量级的数据库，占用资源很少，最初是用于嵌入式的系统，在iOS中使用SQLite，需要加入"libsqlite3.tbd"依赖库并导入头文件。不应该频繁的打开关闭数据库，有可能会影响性能， 应在启动程序时打开数据库，在退出程序是关闭数据库

6.1 封装SQLite3

创建名为HJSQLiteDBManager的类继承自NSObject，导入sqlite3头文件，.h提供操作数据库方法

	#import <Foundation/Foundation.h>


	NS_ASSUME_NONNULL_BEGIN

	@interface HJSQLiteDBManager : NSObject

	/**
	 单利创建Manager

	 @return return value description
 	*/
	+ (instancetype)sharedInstance;

	/**
	 打开数据库
	 */
	+ (void)openDB;

	/**
	 关闭数据库
	 */
	+ (void)closeDB;

	/**
	 删除数据库

	 @return BOOL
	 */
	+ (BOOL)deleteSQliteDB;

	/**
	 创建表

	 @param sql sql语句
	 @return BOOL
	 */
	+ (BOOL)createTableWithSql:(NSString *)sql;

	/**
	 增删改操作
 
	 @param sql sql语句
	 @return BOOL
	 */
	+ (BOOL)operationRecordWithSql:(NSString *)sql;

	/**
	 查询记录

	 @param sql sql语句
	 @return NSArray
	 */
	+ (NSArray *)selectRecordWithSql:(NSString *)sql;
	@end

	NS_ASSUME_NONNULL_END

.m是方法的实现

	//  首先在build phases中去导入libsqlite3.tbd
	//
	//  不应该频繁的打开关闭数据库，有可能会影响性能
	//  在启动程序时打开数据库
	//  在退出程序是关闭数据库
	//  sqlite3_exec() 就是把提到的三个函数结合在了一起：sqlite3_step()， sqlite3_perpare()， 	sqlite3_finalize()
	#import "HJSQLiteDBManager.h"
	#import <sqlite3.h>

	@interface HJSQLiteDBManager ()

	@property (nonatomic, assign) sqlite3 *db;

	@end

	@implementation HJSQLiteDBManager

	#pragma mark - 创建对象
	+ (instancetype)sharedInstance {
	    static HJSQLiteDBManager *manager = nil;
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        manager = [[HJSQLiteDBManager alloc] init];
	    });
	    return manager;
	}
	- (instancetype)init {
	    if (self = [super init]) {

	    }
	    return self;
	}
	#pragma mark - 打开数据库
	+ (void)openDB {
	    sqlite3 *sql = [HJSQLiteDBManager sharedInstance].db;
	    sqlite3_open([[HJSQLiteDBManager getDBpath] UTF8String], &sql);
	    [HJSQLiteDBManager sharedInstance].db = sql;
	}

	+ (NSString *)getDBpath {
	    NSString *documentStr = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
	    NSString *sqlDBFolder = [documentStr stringByAppendingPathComponent:@"SQLiteDB"];
	    if (![[NSFileManager defaultManager] fileExistsAtPath:sqlDBFolder]) {
	        [[NSFileManager defaultManager] createDirectoryAtPath:sqlDBFolder withIntermediateDirectories:YES attributes:nil error:nil];
	    }
	    return [sqlDBFolder stringByAppendingPathComponent:@"dataDemo.sqlite"];
	}

	#pragma mark - 关闭数据库
	+ (void)closeDB {
	    sqlite3_close([HJSQLiteDBManager sharedInstance].db);
	}

	#pragma mark - 删除数据库
	+ (BOOL)deleteSQliteDB {
	    NSString *documentStr = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
	    NSString *sqlDBFolder = [documentStr stringByAppendingPathComponent:@"SQLiteDB"];
	    NSError *error;
	    if ([[NSFileManager defaultManager] fileExistsAtPath:sqlDBFolder]) {
	        [[NSFileManager defaultManager] removeItemAtPath:sqlDBFolder error:&error];
	    }
	    if (error) {
	        return NO;
	    }
	    return YES;
	}

	#pragma mark - 创建表
	+ (BOOL)createTableWithSql:(NSString *)sql {
	    sqlite3 *sqlite = [[HJSQLiteDBManager sharedInstance] db];
	    char *errorMsg = "";
	    if (sqlite3_exec(sqlite, [sql UTF8String], nil, nil, &errorMsg) == SQLITE_OK) {
	        return YES;
	    } else {
	        NSLog(@"创建表失败");
	        return NO;
	    }
	}

	#pragma mark - 查询
	+ (NSArray *)selectRecordWithSql:(NSString *)sql {
	    sqlite3_stmt *stmt;
	    if (sqlite3_prepare_v2([HJSQLiteDBManager sharedInstance].db, [sql UTF8String], -1, &stmt, nil) == SQLITE_OK) {
 	       NSMutableArray *array = [NSMutableArray array];
	        //执行sql语句
	        while (sqlite3_step(stmt) == SQLITE_ROW) {
	            //获取列数
	            int columnCount = sqlite3_column_count(stmt);
	            //获取每一列的值与列明放入字典数组中
	            NSMutableDictionary *dict = [NSMutableDictionary dictionary];
	            for (int i = 0; i < columnCount; i++) {
 	               //获取值
  	              char *value = (char *)sqlite3_column_text(stmt, i);
  	              NSString *valueStr;
  	              if (value != NULL) {
  	                  valueStr = [NSString stringWithUTF8String:value];
  	              }
  	              //获取key
  	              char *key = (char *)sqlite3_column_name(stmt, i);
  	              NSString *keyStr = [NSString stringWithUTF8String:key];
  	              dict[keyStr] = valueStr;
 	           }
 	           [array addObject:dict];
 	       }
	        sqlite3_finalize(stmt);
	        return array;
	    }
	    return @[];
	}


	/**
	 增删改操作记录

	 @param sql sql语句
	 @return BOOL
 	*/
	+ (BOOL)operationRecordWithSql:(NSString *)sql {
	    //对sql执行预编译
	    sqlite3_stmt *stmt;
	    //sqlite3_prepare_v2是执行查询的方法
	    if (sqlite3_prepare_v2([HJSQLiteDBManager sharedInstance].db, [sql UTF8String], -1, &stmt, nil) == SQLITE_OK) {
 	       //执行sql语句
 	       if (sqlite3_step(stmt) == SQLITE_DONE) {
	            sqlite3_finalize(stmt);
 	           return YES;
 	       }
	    }
	    return NO;
	}

	@end

6.2  数据操作示例

封装之后，在"- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions"方法中去调用 "[HJSQLiteDBManager openDB];"打开数据库，然后根据需求调用所封装的操作数据方法进行读写操作。增删改查示例：

	#pragma mark - SQLite3数据操作


	/**
	 添加数据

	 @param sender sender description
	 */
	- (IBAction)sqlAddInfoButtonDidClicked:(UIButton *)sender {
	    NSString *sqlStr = [NSString stringWithFormat:@"CREATE TABLE IF NOT EXISTS %@(userId INT PRIMARY KEY NOT NULL, name CHAR(20) NOT NULL, age INT)",NSStringFromClass([HJPersonModel class])];
	    [HJSQLiteDBManager createTableWithSql:sqlStr];
	    NSString *insetSql = [NSString stringWithFormat:@"INSERT INTO %@ (userId, name) VALUES (2, '小金人')", NSStringFromClass([HJPersonModel class])];
	    [HJSQLiteDBManager operationRecordWithSql:insetSql];
	}
	/**
	 删除数据
 
	 @param sender sender description
	 */
	- (IBAction)sqlDeleteInfoButtonDidClicked:(UIButton *)sender {
	    NSString *deleteSql = [NSString stringWithFormat:@"DELETE FROM %@ WHERE userId = 2", NSStringFromClass([HJPersonModel class])];
	    [HJSQLiteDBManager operationRecordWithSql:deleteSql];
	}
	/**
	 修改数据
 
 	@param sender sender description
	 */
	- (IBAction)sqlUpdateInfoButtonDidClicked:(UIButton *)sender {
	    NSString *updateSql = [NSString stringWithFormat:@"UPDATE %@ SET NAME = '小可爱', AGE = 90 WHERE userId = 2", NSStringFromClass([HJPersonModel class])];
 	   [HJSQLiteDBManager operationRecordWithSql:updateSql];
	}
	/**
	 查询数据
 
	 @param sender sender description
	 */
	- (IBAction)sqlSelectInfoButtonDidClicked:(UIButton *)sender {
	    NSString *selectSql = [NSString stringWithFormat:@"SELECT * FROM %@", NSStringFromClass([HJPersonModel class])];
	    NSArray *array = [HJSQLiteDBManager selectRecordWithSql:selectSql];
	    NSLog(@"%@",array);
	}


7 FMDB

FMDB以OC的方式封装了SQLite的C语言API，减去了冗余的C语言代码，使得API更具有OC的风格，更加的面向对象，相对于Core Data框架更加的轻量级，FMDB还提供了多线程安全的数据库操作方法，在FMDB中有三个重要的概念：

FMDatabase：一个FMDatabase就代表一个SQLite数据库，执行sql语句

FMResultSet：执行查询后的结果集

FMDatabaseQueue：用于在多线程中执行多个查询或更新，安全的

7.1 封装FMDB

FMDB的导入可以直接使用CocoaPods，导入后需要对其进行封装以拥有更加友好的API。创建HJFMDBManager类继承自NSObject，导入FMDatabase和FMDatabaseQueue类，.h文件中提供操作数据库方法

	#import <Foundation/Foundation.h>

	NS_ASSUME_NONNULL_BEGIN

	@interface HJFMDBManager : NSObject

	/**
	 单利manager

	 @return return value description
	 */
	+ (instancetype)sharedInstance;

	/**
	 打开数据库

	 @return BOOL
	 */
	+ (BOOL)openDBForQueue;

	/**
	 关闭数据库
	 */
	+ (void)closeDBForQueue;



	/**
	 删除数据库

	 @return BOOL
	 */
	+ (BOOL)deleteFMDB;


	/**
	 执行增删改建表

	 @param sql sql语句
	 */
	+ (BOOL)executeWithSql:(NSString *)sql;

	/**
	 异步，执行增删改建表

	 @param sql sql语句
	 @param successBlock block
	 */
	+ (void)executeAsynWithSql:(NSString *)sql
                 isSuccess:(void(^)(BOOL isSuccess))successBlock;

	/**
	 执行查询数据
 
	 @param sql sql语句
	 */
	+ (NSArray *)executeQueryWithSql:(NSString *)sql;

	/**
	 异步，执行查询数据

	 @param sql sql语句
	 @param successBlock block
	 */
	+ (void)executeAsynQueryWithSql:(NSString *)sql
                      isSuccess:(void(^)(NSArray *resultArray))successBlock;

	@end

	NS_ASSUME_NONNULL_END
	
.m提供方法的实现

	#import "HJFMDBManager.h"
	#import "fmdb/FMDatabase.h"
	#import "fmdb/FMDatabaseQueue.h"

	@interface HJFMDBManager ()

	@property (nonatomic, strong) FMDatabase *db;
	@property (nonatomic, strong) FMDatabaseQueue *dbQueue;

	@end

	@implementation HJFMDBManager

	#pragma mark 创建manager
	+ (instancetype)sharedInstance {
	    static HJFMDBManager *manager;
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        manager = [[HJFMDBManager alloc] init];
	    });
	    return manager;
	}

	#pragma mark - 打开数据库
	+ (BOOL)openDBForQueue {
	    //获取路径
	    NSString *documentPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
	    //FMDB文件夹
	    NSString *fmdbFolder = [documentPath stringByAppendingPathComponent:@"FMDB"];
	    //判断文件夹是否存在
	    if (![[NSFileManager defaultManager] fileExistsAtPath:fmdbFolder]) {
	        [[NSFileManager defaultManager] createDirectoryAtPath:fmdbFolder withIntermediateDirectories:YES attributes:nil error:nil];
	    }
	    //数据库路径
	    NSString *dbPath = [fmdbFolder stringByAppendingPathComponent:@"fmdb.db"];
	    FMDatabase *db = [FMDatabase databaseWithPath:dbPath];
	    if ([db open]) {
	        [HJFMDBManager sharedInstance].db = db;
	        [HJFMDBManager sharedInstance].dbQueue = [FMDatabaseQueue databaseQueueWithPath:dbPath];
	        return YES;
	    }
 	   return NO;
	}

	#pragma mark - 关闭数据库
	+ (void)closeDBForQueue {
	    [[[HJFMDBManager sharedInstance] dbQueue] close];
	}

	#pragma mark - 删除数据库
	+ (BOOL)deleteFMDB {
	    NSString *documentStr = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
	    NSString *fmdbFolder = [documentStr stringByAppendingPathComponent:@"FMDB"];
	    NSError *error;
	    if ([[NSFileManager defaultManager] fileExistsAtPath:fmdbFolder]) {
	        [[NSFileManager defaultManager] removeItemAtPath:fmdbFolder error:&error];
	    }
	    if (error) {
	        return NO;
	    }
	    return YES;
	}

	#pragma mark - 执行增删改建表
	+ (BOOL)executeWithSql:(NSString *)sql {
	    return [[HJFMDBManager sharedInstance].db executeUpdate:sql];
	}

	#pragma mark - 异步，执行增删改建表
	+ (void)executeAsynWithSql:(NSString *)sql
                     isSuccess:(void(^)(BOOL isSuccess))successBlock {
	    [[[HJFMDBManager sharedInstance] dbQueue] inDatabase:^(FMDatabase * _Nonnull db) {
	        BOOL isSuccess = [db executeUpdate:sql];
	        if (successBlock) {
	            successBlock(isSuccess);
	        }
	    }];
	}

	#pragma mark - 执行查询数据
	+ (NSArray *)executeQueryWithSql:(NSString *)sql {
	    FMResultSet *resultSet = [[HJFMDBManager sharedInstance].db executeQuery:sql];
	    NSMutableArray *array = [NSMutableArray array];
	    while (resultSet.next) {
	        [array addObject:resultSet.resultDictionary];
	    }
	    return array;
	}

	#pragma mark - 异步，执行查询数据
	+ (void)executeAsynQueryWithSql:(NSString *)sql
                      isSuccess:(void(^)(NSArray *resultArray))successBlock {
	    [[[HJFMDBManager sharedInstance] dbQueue] inDatabase:^(FMDatabase * _Nonnull db) {
	        FMResultSet *resultSet = [db executeQuery:sql];
	        NSMutableArray *array = [NSMutableArray array];
	        while (resultSet.next) {
	            [array addObject:resultSet.resultDictionary];
	        }
	        if (successBlock) {
	            successBlock(array);
	        }
	    }];
	}


7.2 数据操作示例

在"- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions"方法中去调用 "[HJFMDBManager openDBForQueue];"打开数据库，然后根据需求调用所封装的操作数据方法进行读写操作。增删改查示例：

	#pragma mark - FMDB数据库操作

	/**
	 增加记录

	 @param sender sender description
	 */
	- (IBAction)fmdbInsertButtonDidClicked:(UIButton *)sender {
	    NSString *creatStr =[NSString stringWithFormat:@"CREATE TABLE IF NOT EXISTS %@(userId INT PRIMARY KEY NOT NULL, name CHAR(20) NOT NULL, age INT)", NSStringFromClass([HJPersonModel class])];
	    [HJFMDBManager executeAsynWithSql:creatStr isSuccess:^(BOOL isSuccess) {
	        if (isSuccess) {
	            NSLog(@"创建表成功");
	        }
	    }];
	    NSString *insertStr = [NSString stringWithFormat:@"INSERT INTO %@ VALUES (2, '小金人', 25)", NSStringFromClass([HJPersonModel class])];
	    [HJFMDBManager executeAsynWithSql:insertStr isSuccess:^(BOOL isSuccess) {
	        if (isSuccess) {
 	           NSLog(@"插入数据成功");
	        }
	    }];
	}
	/**
	 删除记录
 
	 @param sender sender description
	 */
	- (IBAction)fmdbDeleteButtonDidClicked:(UIButton *)sender {
	    NSString *insertStr = [NSString stringWithFormat:@"DELETE FROM %@ WHERE USERID = 2", NSStringFromClass([HJPersonModel class])];
	    [HJFMDBManager executeAsynWithSql:insertStr isSuccess:^(BOOL isSuccess) {
	        if (isSuccess) {
	            NSLog(@"删除数据成功");
 	       }
 	   }];
	}
	/**
	 修改记录
 
	 @param sender sender description
	 */
	- (IBAction)fmdbUpdateButtonDidClicked:(UIButton *)sender {
	    NSString *insertStr = [NSString stringWithFormat:@"UPDATE %@ SET userid = 11, NAME = '小可爱', AGE = 90 WHERE userId = 2", NSStringFromClass([HJPersonModel class])];
 	   [HJFMDBManager executeAsynWithSql:insertStr isSuccess:^(BOOL isSuccess) {
	        if (isSuccess) {
	            NSLog(@"修改数据成功");
	        }
	    }];
	}
	/**
	 查询记录
 
 	@param sender sender description
 	*/
	- (IBAction)fmdbSelectButtonDidClicked:(UIButton *)sender {
	    NSString *selectSql = [NSString stringWithFormat:@"SELECT * FROM %@", NSStringFromClass([HJPersonModel class])];
	    [HJFMDBManager executeAsynQueryWithSql:selectSql
                                 isSuccess:^(NSArray * _Nonnull resultArray) {
	                                     NSLog(@"%@",resultArray);
	                                 }];
	}


8 Realm

Realm是一款可以用于iOS(同样适用于Swift&Objective-C)和Android的跨平台移动数据库。Realm极大地减小了学习成本，没有Core Data与SQLite冗余、复杂的知识和代码，更加的面向对象。Realm还提供了一个轻量级的数据库查看工具"Realm Browser"，可以非常简便的查看数据库当中的内容

8.1 Realm的安装

Realm的安装方法有很多种，使用 Dynamic Framework、CocoaPods等方法都行，这里仅介绍一种

1、下载[最新的Realm版本](https://github.com/realm/realm-cocoa)，并解压，注意选择下载适用的语言，OC应选择下载 "realm-objc"版本

2、选择Xcode 工程的”General”设置项，从文件夹ios/dynamic/、osx/、tvos/或者watchos/中将’Realm.framework’拖曳到”Embedded Binaries”选项中。勾选上Copy items if needed，点击Finish按钮

3、选择 Target 的”Build Settings”中，在”Framework Search Paths”中添加Realm.framework的上级目录；

4、在 iOS、watchOS 或者 tvOS 项目中使用 了Realm，在”Build Phases”中，创建一个新的”Run Script Phase”，将脚本复制到文本框中用来绕过APP商店提交的bug，这在打包通用设备的二进制发布版本时是必须的

	bash "${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/Realm.framework/strip-frameworks.sh"

8.2 Realm的重要概念

RLMRealm：RLMRealm是框架的核心，如同Core Data的托管对象上下文

RLMObject：数据模型，创建自定义的数据模型需要继承自RLMObject

Relationships：在数据模型中声明一个RLMObject类型的属性，就可以创建一个对象关系

Write Transactions：写操作事物，数据库中的创建、删除对象等，都应该在事物中完成

Queries：查询数据库中的信息，可以使用断言、复合查询等进行数据查询

RLMResults：查询结果集，其中包含一系列的RLMObject对象，RLMResults和NSArray类似，我们可以用下标语法来对其进行访问

8.3 Realm数据库的使用

在实际开发中，通常都需要对Core Data、SQLite3、FMDB分装一个单例类。
但是Realm数据库通过[RLMRealm defaultRealm]就可以获取到当前realm对象的一个实例，它本身就是一个单利，所以我们每次在子线程里面不要再去读取我们自己封装持有的realm实例了，直接调用系统的这个方法即可。需要注意的是同一个Realm对象是不支持跨线程操作realm数据库的，在Realm中每个线程都拥有Realm的一个快照，可以同时有任意数目的线程访问同一个 Realm 文件，线程之间不会产生影响

8.3.1 创建数据库

在"- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions"方法中创建数据库

    NSString *documentPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
    NSString *dbFolder = [documentPath stringByAppendingPathComponent:@"RealmDB"];
    if (![[NSFileManager defaultManager] fileExistsAtPath:dbFolder]) {
        [[NSFileManager defaultManager] createDirectoryAtPath:dbFolder withIntermediateDirectories:YES attributes:nil error:nil];
    }
    NSString *filePath = [dbFolder stringByAppendingPathComponent:@"db.realm"];
    RLMRealmConfiguration *config = [RLMRealmConfiguration defaultConfiguration];
    //路径
    config.fileURL = [NSURL URLWithString:filePath];
    //是否只读
    config.readOnly = NO;
    //当前版本
    config.schemaVersion = 4;
    [RLMRealmConfiguration setDefaultConfiguration:config];

8.3.2 创建数据模型

Realm中创建自定义的数据模型需要继承自RLMObject，所以可以创建一个名为HJRealmBaseModel自定义类先继承自RLMObject

	#import <UIKit/UIKit.h>
	#import <Realm/Realm.h>

	NS_ASSUME_NONNULL_BEGIN

	@interface HJRealmBaseModel : RLMObject

	@end

	NS_ASSUME_NONNULL_END

.m文件	

	#import "HJRealmBaseModel.h"

	@implementation HJRealmBaseModel

	@end

创建一个HJRealmPersonModel类和HJRealmCarModel类继承自HJRealmBaseModel，
HJRealmPersonModel对于HJRealmCarModel的关系为一对多，由于Realm数据库不支持集合类型，比如说NSArray，NSMutableArray，NSDictionary，NSMutableDictionary，NSSet，NSMutableSet，所以集合类型的属性是不能直接存储到数据库的。当然Realm里面是有集合的，就是RLMArray，这里面装的都是RLMObject。
解决这个问题，如果是model，就先自己接收一下，然后将数据转换成RLMObject的model，在存储到RLMArray里，然后使用"ignoredProperties"方法忽略掉不能存储的属性

HJRealmCarModel.h

	#import "HJRealmBaseModel.h"

	NS_ASSUME_NONNULL_BEGIN

	@interface HJRealmCarModel : HJRealmBaseModel

	@property (nonatomic, assign) NSInteger carId;
	@property (nonatomic, copy) NSString *carName;

	@end
	//RLM_ARRAY_TYPE宏创建了一个协议，从而允许 RLMArray语法的使用
	RLM_ARRAY_TYPE(HJRealmCarModel)

	NS_ASSUME_NONNULL_END


HJRealmCarModel.m

	#import "HJRealmCarModel.h"

	@implementation HJRealmCarModel

	/**
	 主键

	 @return return value description
	 */
	+ (NSString *)primaryKey {
	    return @"carId";
	}


	/**
	 属性值不能为空

	 @return return value description
	 */
	+ (NSArray<NSString *> *)requiredProperties {
	    return @[@"carId"];
	}

	/**
	 忽略属性

	 @return return value description
	 */
	+ (NSArray *)ignoredProperties {
	    return @[];
	}

	@end


HJRealmPersonModel.h

	#import "HJRealmBaseModel.h"
	#import "HJRealmCarModel.h"

	NS_ASSUME_NONNULL_BEGIN

	@interface HJRealmPersonModel : HJRealmBaseModel

	@property (nonatomic, copy) NSString *name;
	@property (nonatomic, assign) NSInteger userId;
	@property (nonatomic, assign) NSInteger age;
	@property (nonatomic, strong) NSString *address;
	@property (nonatomic, strong) RLMArray<HJRealmCarModel *><HJRealmCarModel> *cars;

	@end
	NS_ASSUME_NONNULL_END


HJRealmPersonModel.m

	#import "HJRealmPersonModel.h"

	@implementation HJRealmPersonModel

	+ (NSString *)primaryKey {
	    return @"userId";
	}

	+ (NSArray<NSString *> *)requiredProperties {
	    return @[@"userId"];
	}

	+ (NSArray *)ignoredProperties {
	    return @[];
	}

	@end


8.3.3 数据的增删改查示例

Realm数据库的操作都应该在事物当中进行

	#pragma mark - Realm数据操作
	/**
	 增加记录
 
	 @param sender sender description
	 */
	- (IBAction)realmInsertButtonDidClicked:(UIButton *)sender {
	    RLMArray<HJRealmCarModel *><HJRealmCarModel> *car;
	    HJRealmCarModel *carModel = [HJRealmCarModel new];
	    carModel.carId = 111111;
	    [car addObject:carModel];
	    HJRealmPersonModel *model = [HJRealmPersonModel new];
	    model.userId = 1;
	    model.cars = car;
	    [[RLMRealm defaultRealm] transactionWithBlock:^{
	        [[RLMRealm defaultRealm] addOrUpdateObject:model];
	    }];
	}
	/**
	 删除记录
 
	 @param sender sender description
	 */
	- (IBAction)realmDeleteButtonDidClicked:(UIButton *)sender {
	    [[RLMRealm defaultRealm] transactionWithBlock:^{
	        RLMResults *results = [HJRealmPersonModel objectsWhere:@"userId = 1"];
	        NSLog(@"%@",results);
	        [[RLMRealm defaultRealm] deleteObjects:results];
	    }];
    
	}
	/**
	 修改记录
 
	 @param sender sender description
 	*/
	- (IBAction)realmUpdateButtonDidClicked:(UIButton *)sender {
	    HJRealmPersonModel *model = [HJRealmPersonModel new];
	    model.userId = 1;
	    model.name = @"小星星";
	    [[RLMRealm defaultRealm] transactionWithBlock:^{
	        [[RLMRealm defaultRealm] addOrUpdateObject:model];
	    }];
	}
	/**
	 查询记录
 
	 @param sender sender description
	 */
	- (IBAction)realmSelectButtonDidClicked:(UIButton *)sender {
	    [[RLMRealm defaultRealm] transactionWithBlock:^{
	        RLMResults *results = [HJRealmPersonModel objectsWhere:@"userId = 1"];
	        NSLog(@"%@",results.firstObject);
	    }];
	}


>总结
经过上面的分析
plist文件主要用于不经常修改、数据量小的数据
偏好设置主要用于检查版本是否更新、是否启动引导页、自动登录、版本号等程序设置归
档解归档主要是归档之后的文件时加密的，用于一些数据量小的数据，数据库操作比较笨拙
沙盒路径可以提高程序的体验度，为用户节约数据流量，主要在用户阅读书籍、听音乐、看视频等
Core Data提供了对象关系的映射功能，使得能够将OC对象转换成数据，将数据库中的数据还原成OC对象，在转换的过程中不需要编写任何的SQL语句
SQLite3是轻量级的数据库，占用资源很少，需要大量的SQL语句
FMDB是对SQLite3的进一步封装，减去了C语言风格，更加的面向对象，同样需要大量的SQL语句
Realm本质上也是一个嵌入式数据库，更加的轻量级，支持跨平台，没有Core Data与SQLite冗余、复杂的知识和代码，更加的面向对象，学习成本更小
