# 技术概述
  语言：Python              
  主要用第三方库：pygame          
  主要模块使用：
  函数surface返回新的可用于绘画与传送surface对象
  函数init用于在游戏进入主循环前调用自动初始化其他所有模块
  函数pygame.locals：
  
  # 导入库
    # -*- coding: utf-8 -*-
    from collections import deque
    import pygame, sys, os
    from pygame.locals import *
    
# 按钮设置
    class Button(object):
        def __init__(self, buttonUpImage, buttonDownImage, pos):
            self.buttonUp = pygame.image.load(buttonUpImage).convert_alpha()
            self.buttonDown = pygame.image.load(buttonDownImage).convert_alpha()
            self.pos = pos
            #pygame.image.load:载入图片
            #pygame.image.load.convert:在显示图像时转换格式提供blit速度（无法显示透明效果）
            #pygame.image.load.convert_alpha:在显示图像时转换格式提供blit速度（可以显示透明效果）

  
# 鼠标设置
        def inButtonRange(self):
            mouseX, mouseY = pygame.mouse.get_pos()
            x, y = self.pos
            w, h = self.buttonUp.get_size()
            inX = x - w / 2 < mouseX < x + w / 2
            inY = y - h / 2 < mouseY < y + h / 2
            return inX and inY
            #pygame.mouse.get_pos()函数：获取鼠标的位置
    
# 按钮
        def show(self, screen):
            w, h = self.buttonUp.get_size()
            x, y = self.pos
            if self.inButtonRange():
                screen.blit(self.buttonDown, (x - w / 2, y - h / 2))
            else:
                screen.blit(self.buttonUp, (x - w / 2, y - h / 2))
                #根据鼠标位置变换样式
                #screen.blit( ) 在屏幕上创建控件
    
# 地图元素绘制
    class Sokoban:
        def __init__(self):
            #地图绘制——.：空白  #：墙  @：玩家  $：箱子  *：终点  &：到终点的箱子
            self.level = [list('...#######' + \
                               '####.....#' + \
                               '#..*...$.#' + \
                               '#..*..#.##' + \
                               '#.#####.#.' + \
                               '#.#...#.#.' + \
                               '#...$.#.#.' + \
                               '###$..#.#.' + \
                               '#...$...#.' + \
                               '#.@.....#.' + \
                               '#########.'),
                          list('.#######....' + \
                               '##.....##...' + \
                               '#..#.$..#...' + \
                               '#.$...#.#...' + \
                               '#..#.$..###.' + \
                               '#....#....##' + \
                               '######..*..#' + \
                               '.....#.#*..#' + \
                               '.....#..*..#' + \
                               '.....#.$*@.#' + \
                               '.....#######'),]
            # 地图大小（宽高）
            self.w = [10, 12]
            self.h = [11, 11]
            # 玩家位置
            self.man = [92, 117]
            # 箱子个数
            self.boxCnt = [4, 4]
            # 通关要求
            self.boxInPositionCnt = [2, 0]
    
  # 在界面中判断“空白”,“玩家”与“终点”
        def toBox(self, chapter, index):
            if self.level[chapter][index] == '.' or self.level[chapter][index] == '@':
                self.level[chapter][index] = '$'
            else:
                self.level[chapter][index] = '&'
    
        def toMan(self, chapter, index):
            if self.level[chapter][index] == '.' or self.level[chapter][index] == '$':
                self.level[chapter][index] = '@'
            else:
                self.level[chapter][index] = '+'
    
        def toFloor(self, chapter, index):
            if self.level[chapter][index] == '@' or self.level[chapter][index] == '$':
                self.level[chapter][index] = '.'
            else:
                self.level[chapter][index] = '*'
    
 # 移动计算(上下左右键控制玩家移动)
        def offset(self, d, width):
            d4 = [-1, -width, 1, width]
            m4 = ['l', 'u', 'r', 'd']
            return d4[m4.index(d.lower())]
    
# 地图绘制，判断不同符号对应的事物
        def draw(self, screen, chapter, skin):
            w = skin.get_width() / 4
            for i in range(0, self.w[chapter]):
                for j in range(0, self.h[chapter]):
                    if self.level[chapter][j * self.w[chapter] + i] == '#':
                        screen.blit(skin, (i * w, j * w), (0, 2 * w, w, w))
                    elif self.level[chapter][j * self.w[chapter] + i] == '.':
                        screen.blit(skin, (i * w, j * w), (0, 0, w, w))
                    elif self.level[chapter][j * self.w[chapter] + i] == '@':
                        screen.blit(skin, (i * w, j * w), (w, 0, w, w))
                    elif self.level[chapter][j * self.w[chapter] + i] == '$':
                        screen.blit(skin, (i * w, j * w), (2 * w, 0, w, w))
                    elif self.level[chapter][j * self.w[chapter] + i] == '*':
                        screen.blit(skin, (i * w, j * w), (0, w, w, w))
                    elif self.level[chapter][j * self.w[chapter] + i] == '+':
                        screen.blit(skin, (i * w, j * w), (w, w, w, w))
                    elif self.level[chapter][j * self.w[chapter] + i] == '&':
                        screen.blit(skin, (i * w, j * w), (2 * w, w, w, w))
    
 # 玩家移动轨迹
        def move(self, chapter, op):
            h = self.offset(op, self.w[chapter])
            if self.level[chapter][self.man[chapter] + h] == '.' or self.level[chapter][self.man[chapter] + h] == '*':
                self.toMan(chapter, self.man[chapter] + h)
                self.toFloor(chapter, self.man[chapter])
                self.man[chapter] += h
                return 0
            #没有遇到箱子和墙体——自由移动
            elif self.level[chapter][self.man[chapter] + h] == '&' or self.level[chapter][self.man[chapter] + h] == '$':
                if self.level[chapter][self.man[chapter] + 2 * h] == '.' or self.level[chapter][
                    self.man[chapter] + 2 * h] == '*':
            # 遇到箱子——实行推动
                    if self.level[chapter][self.man[chapter] + h] == '&':
                        self.boxInPositionCnt[chapter] -= 1
                    self.toBox(chapter, self.man[chapter] + 2 * h)
                    self.toMan(chapter, self.man[chapter] + h)
                    self.toFloor(chapter, self.man[chapter])
                    self.man[chapter] += h
            # 箱子到达终点——目标数加一
                    if self.level[chapter][self.man[chapter] + h] == '&':
                        self.boxInPositionCnt[chapter] += 1
                    return 1
                else:
                    return -1
            else:
                return -1
    
 # 回到上一步
        def remove(self, chapter, op):
            d = op[0]
            flag = op[1]
            h = self.offset(d, self.w[chapter])
            #记录上一步移动轨迹
            if flag == 1:
            # 人推着箱子移动——将人与箱子还原
                if self.level[chapter][self.man[chapter] + h] == '&':
                    self.boxInPositionCnt[chapter] -= 1
                self.toBox(chapter, self.man[chapter])
                self.toMan(chapter, self.man[chapter] - h)
                self.toFloor(chapter, self.man[chapter] + h)
                self.man[chapter] -= h
                if self.level[chapter][self.man[chapter] + h] == '&':
                    self.boxInPositionCnt[chapter] += 1
            # 只有人移动——还原人的位置
            elif flag == 0:
                self.toMan(chapter, self.man[chapter] - h)
                self.toFloor(chapter, self.man[chapter])
                self.man[chapter] -= h
    
    
 # 主界面
    def showGameInterface(screen, interface, startGame, gameTips):
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == MOUSEBUTTONDOWN:
                if startGame.inButtonRange():
                    return 1
                elif gameTips.inButtonRange():
                    return 2
        screen.blit(interface, (0, 0))
        startGame.show(screen)
        gameTips.show(screen)
        return 0
    #event.type 属性返回哪种事件类型被触发
    #event.key属性返回哪种按键被触发
    
# 关卡选择
    def showChapterInterface(screen, interface, chapter1, chapter2, prevInterface):
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == MOUSEBUTTONDOWN:
                if chapter1.inButtonRange():
                    return 1
                elif chapter2.inButtonRange():
                    return 2
                elif prevInterface.inButtonRange():
                    return 4
        screen.blit(interface, (0, 0))
        chapter1.show(screen)
        chapter2.show(screen)
        prevInterface.show(screen)
        return 0
    
    
 # 胜利界面
    def showWinInterface(screen, interface, nextChapter, returnToChoose, flag):
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == MOUSEBUTTONDOWN:
                if nextChapter.inButtonRange():
                    return 1
                elif returnToChoose.inButtonRange():
                    return 2
        screen.blit(interface, (0, 0))
        if flag:
            nextChapter.show(screen)
        returnToChoose.show(screen)
        return 0
    
# 加载图片资源
    def main():
        pygame.init()
        try:#模型来源：百度推箱子吧
            gameicon = pygame.image.load("图标.jpg")
            pygame.display.set_icon(gameicon)
            bgm = ["Laura Shigihara - Moongrains.mp3","Laura Shigihara - Moongrains.mp3"]
            pygame.mixer.music.set_volume(0.5)
            screen = pygame.display.set_mode((400, 400))
            button1 = Button('开始游戏.png', '开始游戏2.png', (200, 300))
            button2 = Button('游戏说明.png', '游戏说明2.png', (200, 350))
            buttonc1 = Button('第一关.png', '第一关2.png', (200, 175))
            buttonc2 = Button('第二关.png', '第二关2.png', (200, 225))
            buttonmain = Button('返回.png', '返回2.png', (200, 350))
            buttonnext = Button('下一关.png', '下一关2.png', (100, 350))
            buttonret1 = Button('选关卡.png', '选关卡2.png', (300, 350))
            buttonret2 = Button('选关卡.png', '选关卡2.png', (200, 350))
            interface = pygame.image.load("封面.png")
            gametips = pygame.image.load("游戏说明具体.png")
            choosechapter = pygame.image.load("二号主界面.png")
            chapterpass = pygame.image.load("恭喜通关.png")
            skinfilename = os.path.join('下的模型.png')
            skin = pygame.image.load(skinfilename)
            skin = skin.convert()
        except pygame.error as msg:
            raise (SystemExit(msg))
    
# 运行游戏
        pygame.display.set_caption('推箱子')
        clock = pygame.time.Clock()
        # 按键立刻触发
        pygame.key.set_repeat(200, 50)
        while True:
            clock.tick(60)
# 记录按键
            flag = showGameInterface(screen, interface, button1, button2)
            if flag == 1:
                while True:
                    clock.tick(60)
                    #记录按键
                    chapter = showChapterInterface(screen, choosechapter, buttonc1, buttonc2, buttonmain) - 1
                    if chapter == 3:
                        break
                    if chapter == -1:
                        pygame.display.update()
                        continue
                    skb = Sokoban()
# 设置关卡窗口
                    screen.fill(skin.get_at((0, 0)))
                    skb.draw(screen, chapter, skin)
                    # 播放音乐
                    pygame.mixer.music.load(bgm[chapter])
                    pygame.mixer.music.play(-1)
                    dq = deque([])
                    while True:
                        clock.tick(60)
                        retGameInterface = False
                        for event in pygame.event.get():
                            if event.type == QUIT:
                                pygame.quit()
                                sys.exit()
                            elif event.type == KEYDOWN:
# Ese键退出
                                if event.key == K_ESCAPE:
                                    retGameInterface = True
                                    break
# 上下左右按键控制
                                elif event.key == K_LEFT:
                                    flag = skb.move(chapter, 'l')
                                    dq.append(['l', flag])
                                    skb.draw(screen, chapter, skin)
                                elif event.key == K_UP:
                                    flag = skb.move(chapter, 'u')
                                    dq.append(['u', flag])
                                    skb.draw(screen, chapter, skin)
                                elif event.key == K_RIGHT:
                                    flag = skb.move(chapter, 'r')
                                    dq.append(['r', flag])
                                    skb.draw(screen, chapter, skin)
                                elif event.key == K_DOWN:
                                    flag = skb.move(chapter, 'd')
                                    dq.append(['d', flag])
                                    skb.draw(screen, chapter, skin)
# Backspace撤销操作
                                elif event.key == K_BACKSPACE:
                                    if len(dq) > 0:
                                        # 如果队列不为空则取出队尾对象
                                        op = dq.pop()
# 还原操作
                                        skb.remove(chapter, op)
                                        skb.draw(screen, chapter, skin)
                                if len(dq) > 50:
                                    #限制可返回步数为50，超过上限则从头删除记录
                                    dq.popleft()
# 到达终点箱子数量=要求箱子数，则过关
                        if skb.boxInPositionCnt[chapter] == skb.boxCnt[chapter]:
                            clock.tick(60)
                            pygame.mixer.music.stop()
                            if chapter == 0 :
                                win = showWinInterface(screen, chapterpass, buttonnext, buttonret1, 1)
                            elif chapter == 1:
                                win = showWinInterface(screen, chapterpass, buttonnext, buttonret2, 0)
# 记录点击按钮
                            # 点击下一关
                            if win == 1:
                                # 清空队列
                                dq.clear()
# 如果不是最后一关跳到下一关
                                if chapter < 1:
                                    chapter += 1
# 绘制下一关的关卡地图布局
                                    screen.fill(skin.get_at((0, 0)))
                                    skb.draw(screen, chapter, skin)
# 播放音乐
                                    pygame.mixer.music.load(bgm[chapter])
                                    pygame.mixer.music.play(-1)
                                pygame.display.update()
                                continue
# 退出游戏循环
                            elif win == 2:
                                retGameInterface = True
# 退出游戏关闭所有音乐
                        if retGameInterface == True:
                            pygame.mixer.music.stop()
                            pygame.display.update()
                            break
                        pygame.display.update()
# 游戏说明
            elif flag == 2:
                clock.tick(60)
                retGameInterface = False
                while True:
                    for event in pygame.event.get():
                        if event.type == QUIT:
                            pygame.quit()
                            sys.exit()
                        elif event.type == KEYDOWN:
                            if event.key == K_ESCAPE:
                                retGameInterface = True
                                break
                    screen.blit(gametips, (0, 0))
                    pygame.display.update()
                    if retGameInterface == True:
                        break
            pygame.display.update()
    
# 运行游戏
    if __name__ == '__main__':
    main()
