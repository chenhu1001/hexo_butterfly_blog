---
title: iOS协议
date: 2016-05-17 20:04:29
categories: iOS
tags: [iOS]
---
iOS中协议虽然OC基础中有讲过，但一直只是表面的认识，今天在做项目时，有了更深入一些的认识。在请求网络数据并解析xml时协议就体现出它的价值了。解析数据后，最后会通过协议进行加载解析后的数据。  
下面以例子说明：  
示例一、
* 1、声明协议

```
@protocol updateSearchDataDelegate <NSObject>

-(void)loadDataForShow;
@end

@interface AuctionDetailXmlParser : NSObject <NSXMLParserDelegate, ASIHTTPRequestDelegate>

//定义协议
@property (weak,nonatomic) id<updateSearchDataDelegate> delegate;

........

@end
```

可以用协议delegate调用loadDataForShow方法。

* 2、当其它类引入协议时，则需要实现协议里面的方法。

```
@interface AuctionDetailsViewController : UITableViewController <updateSearchDataDelegate>

......

@end

@implementation AuctionDetailsViewController

- (void)getAuctionDetails {
    auctionDetailXmpParser = [[AuctionDetailXmlParser alloc] init];

   //设置代理，只有设置协议才会调用协议中方法
    auctionDetailXmpParser.delegate = self; 
    [auctionDetailXmpParser getAuctionDetails:strXml Action:actionName];  
}

- (void)loadDataForShow {

  ........
}

@end
```

示例二、（当请求另一个viewController时，通过协议可不通过new来请求该viewController方法）

```
#import <UIKit/UIKit.h>
#import "RequestVO.h"
//协议
@protocol sortDataDelegate <NSObject>
@required
- (void) loadDataShow : (RequestVO *)requestParams;
@end

@interface SortViewController : UITableViewController
@property (weak,nonatomic) id<sortDataDelegate> delegate;

@end
```

```
//
//  SortViewController.m
//  BookReader
//
//  Created by Dwen on 13-1-29.
//
//

#import "SortViewController.h"
#import "Utils.h"

@interface SortViewController ()

@end

@implementation SortViewController
@synthesize popover,sortArry;
@synthesize sortStr;
@synthesize specialCode;
@synthesize delegate;

- (id)initWithStyle:(UITableViewStyle)style
{
    self = [super initWithStyle:style];
    if (self) {
         sortArry = [[NSMutableArray alloc] initWithObjects:CLOSE_COST_ASC,CLOSE_COST_DESC,LOT_ASC,LOT_DESC, nil];
    }
    return self;
}

#pragma mark - Table view data source

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
   return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return 4;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *CellIdentifier = @"Cell";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier];
    }
    cell.textLabel.text =@"test";
    cell.textLabel.font = [UIFont fontWithName:@"Arial" size:18];
    return cell;
}

#pragma mark - Table view delegate

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    RequestVO *requestVO = [[RequestVO alloc] init];
    [delegate loadDataShow:requestVO];//****代理方法解决调用Controller问题
}

@end
```

```
#import <UIKit/UIKit.h>

@interface CatalogViewController : UIViewController <sortDataDelegate>

@end


@interface CatalogViewController ()

@end

@implementation CatalogViewController

//排序
- (IBAction)sortAction:(id)sender {
    sortVC = [[SortViewController alloc] initWithStyle:UITableViewStylePlain];
    sortVC.contentSizeForViewInPopover = CGSizeMake(240, 176);
    popover = [[UIPopoverController alloc] initWithContentViewController:sortVC];
    [popover presentPopoverFromRect:[self.sortBtn bounds] inView:self.sortBtn permittedArrowDirections:UIPopoverArrowDirectionAny animated:YES];
    sortVC.delegate = self;//****该delegate一定要设
}

- (void)loadDataShow:(RequestVO *)requestParams{

}

@end
```
