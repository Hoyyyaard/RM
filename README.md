# STATE MACHINE

code below

```cpp
#include <map>
#include "iostream"

/* Private variables ------------------------------------------------------*/

/*定义客户端感兴趣的接口（环境类）*/
class CONTEXT;

/*状态抽象类*/
class STATE
{
public:
STATE(uint8_t cur_state) : _state(cur_state) {};
~STATE() {};

/*功能函数*/
virtual void init() = 0;
virtual bool run() = 0;
virtual void exit() = 0;
virtual bool checkState() = 0;      //检测电机是否到达目标位置

private:
uint8_t _state;
};



/*状态类*/
enum MOTION{
none = 0,
lifting ,
extending,
overturning,
allmotion

};


/*初始状态*/
class NOMOTION final : public STATE{
public:
NOMOTION(uint8_t state = none) : STATE(state){};
~NOMOTION();
void init() override{
std::cout << "motion : none init" << std::endl;
}

bool run() override {
std::cout << "motion : none" << std::endl;
}

void exit() override{
std::cout << "motion : none motion exit" << std::endl;
}

bool checkState() override{
std::cout << "motion :check state" << std::endl;
return true;
}

};

/*抬升状态*/
class LIFTING final : public STATE {
public:
LIFTING(uint8_t state = lifting) : STATE(state){};
~LIFTING();
void init() override{
std::cout << "motion : lifting init" << std::endl;
}

bool run() override {
std::cout << "motion : lifting" << std::endl;
}

void exit() override{
std::cout << "motion : lifting exit" << std::endl;
}

bool checkState() override{
std::cout << "motion :lifting check state" << std::endl;
return true;
}

};

/*伸出状态*/
class EXTENDING final : public STATE{
public:
EXTENDING(uint8_t state = extending) : STATE(state){};
~EXTENDING();
void init() override{
std::cout << "motion : extending init" << std::endl;
}

bool run() override{
std::cout << "motion : extending" << std::endl;
}
void exit() override{
std::cout << "motion : extending exit" << std::endl;
}

bool checkState() override{
std::cout << "motion :extending check state" << std::endl;
return true;
}
};

/*翻转状态*/
class OVERTURNING final : public STATE{
public:
OVERTURNING(uint8_t state = overturning) : STATE(state){};
~OVERTURNING();
void init() override{
std::cout << "motion : overturning init" << std::endl;
}

bool run () override{
std::cout << "motion : overturning" << std::endl;
}
void exit() override{
std::cout << "motion : overturning exit" << std::endl;
}

bool checkState() override{
std::cout << "motion :overturning check state" << std::endl;
return true;
}
};

/*三个基本动作同时*/
class ALLMOTION final : public STATE{
public:
ALLMOTION(uint8_t state = allmotion) : STATE(state){};
~ALLMOTION();
void init() override{
std::cout << "motion : all motion init" << std::endl;
}

bool run () override{
std::cout << "motion : all motion" << std::endl;
}
void exit() override{
std::cout << "motion : all motion exit" << std::endl;
}

bool checkState() override{
std::cout << "motion :all motion check state" << std::endl;
return true;
}
};

/*环境类*/
class CONTEXT
{
public:
CONTEXT() {};
~CONTEXT() {};

uint8_t transitionTo(uint8_t state) {
if (state == _cur_state) {
return 99;  //state is running
}

auto iter = _state_list.find(_cur_state);
if (iter == _state_list.end()) {
return 100; //state list doesnot include _cur_state
}
_state_list[_cur_state]->exit();
_cur_state = 0;

iter = _state_list.find(state);
if (iter == _state_list.end()) {
return 101; //state list doesnot include state
}
_state_list[state]->init();
_cur_state = state ;

return 0;
}

uint8_t getCurstate(){
return _cur_state;
}

std::map<uint8_t, STATE*>_state_list;

private:
uint8_t _cur_state = 0;


};

/*总控类*/
class ENGINEER final : public CONTEXT{
public:
ENGINEER(){
CONTEXT::_state_list.clear();
CONTEXT::_state_list.insert({none,  _NOMOTION});
CONTEXT::_state_list.insert({lifting,  _LIFTING});
CONTEXT::_state_list.insert({extending,  _EXTENDING});
CONTEXT::_state_list.insert({overturning,  _OVERTURNING});
CONTEXT::_state_list.insert({allmotion,  _ALLMOTION});
}

void bigMineral(){
switch (this->getCurstate()) {
case none:
/*切换到抬升模式*/
this->transitionTo(lifting);
_state_list[getCurstate()]->run();

break;
case lifting:
if(_state_list[getCurstate()]->checkState()){
/*如果抬升到位 切换到伸出*/
this->transitionTo(extending);
}
_state_list[getCurstate()]->run();
break;

case extending:
if(_state_list[getCurstate()]->checkState()){
this->transitionTo(overturning);
}
_state_list[getCurstate()]->run();
break;
default:break;

}
}

void smallMineral(){
switch (this->getCurstate()) {
case none:
this->transitionTo(allmotion);
_state_list[getCurstate()]->run();
break;
case allmotion:
break;
}
}

private:
STATE *_NOMOTION = new NOMOTION();
STATE *_LIFTING = new LIFTING();
STATE *_EXTENDING = new EXTENDING();
STATE *_OVERTURNING = new OVERTURNING();
STATE *_ALLMOTION = new ALLMOTION();
};

ENGINEER engineerControl;

int main(){
std::cout << "welcome to state machine" << std::endl;
while(1){
engineerControl.smallMineral();
}


}
```