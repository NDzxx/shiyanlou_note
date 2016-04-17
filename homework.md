# 实验楼大作业

本周只看完了第15章，其余慢慢看吧，不过内容够完成报告了 除了需求说的项目课之外，我想输入也可以按语言分成多种
```

       -》项目课                -》中文
课程    -》评估课       命令系统  -》英文
```

个人对继承的理解：这个项目因为实现也是自己写，所以继承最大的作用体现不出来。继承最大作用是偷懒 你只需要提供一个接口，比如
```C++
class course
{
 public:
    // 写成纯虚函数，类似java接口，这样所有的函数都可以让调用人来写了
    virtual void PrintHelpInfo() = 0;
}
```
贴上实验代码 
##course.h
```C++
#ifndef __COURSE_H_
#define __COURSE_H_
#include <string>
#include <list>
#include <map>
#include <vector>
#include <fstream>
#include <iostream>
#include <sstream>
#include <algorithm>
#include <memory>
using namespace std;
typedef unsigned int OBJID;
extern OBJID g_idCourse;
typedef enum sortEnum
{
    sortFromLesstoMore,
    sortFromMoretoLess,
};
typedef enum courseTypeEnum
{
    xm_course = 1,
    pg_course = 2,
    //后续添加其他课程类型
};
class CCourseInfo
{
    friend void read(std::istream& is, CCourseInfo& info);
public:
    CCourseInfo(string courseName);//载入文件内容的课程
    CCourseInfo(istream &stream);
    CCourseInfo(const CCourseInfo& rInfo);
    virtual ~CCourseInfo();
    virtual void init(string courseName);
    CCourseInfo & operator=(const CCourseInfo& rInfo);//拷贝赋值运算符
    bool operator==(const CCourseInfo& rInfo) const;//相等运算符
    operator string() const;
    //重载输入输出运算符，只能用友元函数
    friend ostream &operator<<(ostream &os, const CCourseInfo &rInfo);
    friend istream &operator>>(istream &is, CCourseInfo &rInfo);
public:
    inline const OBJID GetCourseId() const { return m_idCourse; }
    inline const string& GetCourseName() const{ return m_sCouseName; };
    inline void setCourseName(const string &rsName){ m_sCouseName = rsName; };
    virtual void PrintInfo(stringstream &ss) const;
    inline  void SetHomework(string homework);
    virtual void AnnounceHomework(stringstream &ss) const;
protected:
    CCourseInfo() = default;
private:
    OBJID  m_idCourse;
    string m_sCouseName;
protected:
    string m_shomework="暂时没有作业";
};
class CXMCourse :public CCourseInfo//项目课
{
public:
    CXMCourse(string courseName);
    ~CXMCourse() = default;
    void SetXMCourseInfo(string courseType,string tag);
    virtual void PrintInfo(stringstream &ss) const override;
    virtual void AnnounceHomework(stringstream &ss) const override;
private:
    string m_sCourseType;
    string m_stag;
};
class CPGCourse :public CCourseInfo//评估课
{
public:
    CPGCourse(string courseName);
    ~CPGCourse() = default;
    void SetPGCourseInfo(int leftTime);
    virtual void PrintInfo(stringstream &ss) const override;
    virtual void AnnounceHomework(stringstream &ss) const override;
private:
    int m_leftTime;//评估课剩余完成时间
};
class CCourseMgr
{
public:
    CCourseMgr() = default;
    ~CCourseMgr();
    //简单工厂模式建立课程
    bool CreateCourse(courseTypeEnum type,string strCourseName,string strHomeWork);
    CCourseMgr(CCourseInfo infoArr[], int arrlen);
    CCourseMgr(const vector<CCourseInfo> &rCourseVec);
    CCourseMgr(const CCourseMgr &rCourseMgr);//拷贝构造函数
    CCourseMgr& operator=(CCourseMgr& rMgr);//拷贝赋值运算符
    shared_ptr<CCourseInfo> operator[] (OBJID id);
public:
    inline int GetCourseNum() { return m_mapCourse.size(); };
    bool AddCourse(const CCourseInfo &rCourseInfo);
    bool AddCourse(const string &sCourseName);
    bool DelCourse(OBJID idCourse);
    void DelLatestCourse();
    bool DelCourseByName(const string &sCourseName);
    void PrintCourseList(stringstream &ss) const;
    void FindLongNameCourse(stringstream &ss);
    void SaveCourse(fstream &fs);
    void SortCourse(sortEnum sortType, vector<string> &vecSort);
    bool SearchCourseByName(string szName, vector<string> &vecCorrect);
private:
    typedef map<OBJID/*id*/, shared_ptr<CCourseInfo>> COURSE_MAP;
    COURSE_MAP m_mapCourse;
};
#endif
```  
##course.cpp  
```c++
#include "course.h"
extern OBJID g_idCourse = 0;
void read(std::istream& is, CCourseInfo& info)
{
    is >> info.m_sCouseName;
    ++g_idCourse;
    info.m_idCourse = g_idCourse;
}
CCourseInfo::CCourseInfo(string courseName)
{
    m_sCouseName = courseName;
    ++g_idCourse;
    m_idCourse = g_idCourse;
}
CCourseInfo::CCourseInfo(istream &stream)
{
    read(stream, *this);
}
CCourseInfo::CCourseInfo(const CCourseInfo& rInfo)
{
    m_sCouseName = rInfo.m_sCouseName;//拷贝构造函数只复制课程名
    m_idCourse = rInfo.m_idCourse;
}
CCourseInfo::~CCourseInfo()
{
}
void CCourseInfo::init(string courseName)
{
    m_sCouseName = courseName;
    ++g_idCourse;
    m_idCourse = g_idCourse;
}
CCourseInfo& CCourseInfo::operator=(const CCourseInfo& rInfo)
{
    if (this == &rInfo)
        return *this;
    m_sCouseName = rInfo.m_sCouseName;
    m_idCourse = rInfo.m_idCourse;
    return *this;
}
bool CCourseInfo::operator==(const CCourseInfo& rInfo) const
{
    return (this->m_idCourse == rInfo.m_idCourse) 
    && (this->m_sCouseName == rInfo.m_sCouseName);
}
CCourseInfo::operator string() const
{
    string strCourse("id:");
    strCourse += m_idCourse;
    strCourse += "courseName:";
    strCourse += m_sCouseName;
    return strCourse;
}
ostream &operator<<(ostream &os, const CCourseInfo &rInfo)
{
    os << "courseId:os" << rInfo.m_idCourse << "+" 
    << "courseName:" << rInfo.m_sCouseName << endl;
    return os;
}
istream &operator>>(istream &is, CCourseInfo &rInfo)
{
    return is >> rInfo.m_idCourse >> rInfo.m_sCouseName;
}
void CCourseInfo::PrintInfo(stringstream &ss) const
{
    ss << "course id:" << m_idCourse << endl;
    ss << "course name:" << m_sCouseName << endl;
}
void CCourseInfo::SetHomework(string homework)
{
    m_shomework = homework;
}
void CCourseInfo::AnnounceHomework(stringstream &ss) const
{
    ss <<"实验楼爬爬:课程公告请注意啦！重要的事情要说三遍！说三遍！三遍！" << endl;
}
CXMCourse::CXMCourse(string courseName)
{
    CCourseInfo::init(courseName);
}
void CXMCourse::SetXMCourseInfo(string courseType, string tag)
{
    m_sCourseType = courseType;
    m_stag = tag;
}
void CXMCourse::PrintInfo(stringstream &ss) const
{
    CCourseInfo::PrintInfo(ss);
    ss << "course type:" << m_sCourseType << endl;
    ss << "course tag:" << m_stag << endl;
}
void CXMCourse::AnnounceHomework(stringstream &ss) const
{
    CCourseInfo::AnnounceHomework(ss);
    ss << m_shomework << endl;
}
CPGCourse::CPGCourse(string courseName)
{
    CCourseInfo::init(courseName);
}
void CPGCourse::SetPGCourseInfo(int leftTime)
{
    m_leftTime = leftTime;
}
void CPGCourse::PrintInfo(stringstream &ss) const
{
    CCourseInfo::PrintInfo(ss);
    ss << "评估课剩余完成时间:" << m_leftTime << endl;
}
void CPGCourse::AnnounceHomework(stringstream &ss) const
{
    CCourseInfo::AnnounceHomework(ss);
    ss << m_shomework << endl;
}
bool  CCourseMgr::CreateCourse(courseTypeEnum type, \
string strCourseName, string strHomeWork)
{
    try
    {
        shared_ptr<CCourseInfo> pRef = nullptr;
        switch (type)
        {
        case xm_course:
        {
            pRef = make_shared<CXMCourse>(strCourseName);
            pRef->SetHomework(strHomeWork);
            break;
        }
        case pg_course:
        {
            pRef = make_shared<CPGCourse>(strCourseName);
            pRef->SetHomework(strHomeWork);
            break;
        }
        default:
            break;
        }
        if (pRef != nullptr)
        {
            auto ret = m_mapCourse.insert(make_pair(pRef->GetCourseId(), pRef));
            if (ret.second)
            {
                return true;
            }
        }
        return false;
    }
    catch (const exception& e)
    {
        cout << "file:" << __FILE__ << "line:" << __LINE__ << endl;
        cout << e.what() << endl;
        return false;
    }
}
CCourseMgr::CCourseMgr(CCourseInfo infoArr[], int arrlen)
{
    m_mapCourse.clear();
    for (int i = 0; i > arrlen; ++i)
    {
        shared_ptr<CCourseInfo> pRef = make_shared<CCourseInfo>(infoArr[i]);
        m_mapCourse.insert(make_pair(infoArr[i].GetCourseId(), pRef));
    }
}
CCourseMgr::CCourseMgr(const vector<CCourseInfo> &rCourseVec)
{
    m_mapCourse.clear();
    for (const auto &rInfo : rCourseVec)
    {
        shared_ptr<CCourseInfo> pRef = make_shared<CCourseInfo>(rInfo);
        m_mapCourse.insert(make_pair(rInfo.GetCourseId(), pRef));
    }
}
CCourseMgr::CCourseMgr(const CCourseMgr &rCourseMgr)
{
//转1手，防止调用的是this,丢失了vec内的内容
    COURSE_MAP tmpMap = rCourseMgr.m_mapCourse;
    m_mapCourse.clear();
    m_mapCourse = tmpMap;
}
CCourseMgr& CCourseMgr::operator=(CCourseMgr& rMgr)
{
    COURSE_MAP tmpMap = rMgr.m_mapCourse;
    m_mapCourse.clear();
    m_mapCourse = tmpMap;
    return *this;
}
shared_ptr<CCourseInfo> CCourseMgr::operator[](OBJID id)
{
    try
    {
        auto iter_find = m_mapCourse.find(id);
        if (iter_find != m_mapCourse.end())
            return iter_find->second;
        else
            return nullptr;
    }
    catch (const exception& e)
    {
        cout << "file:" << __FILE__ << "line:" << __LINE__ << endl;
        cout << e.what() << endl;
        return nullptr;
    }
}
CCourseMgr::~CCourseMgr()
{
    m_mapCourse.clear();//内部变量由智能指针自动释放
}
bool CCourseMgr::AddCourse(const CCourseInfo &rCourseInfo)
{
    auto tmpInfo = rCourseInfo;
    shared_ptr<CCourseInfo> pRef(&tmpInfo);
    auto ret = m_mapCourse.insert(make_pair(rCourseInfo.GetCourseId(), pRef));
    if (ret.second)
    {
        return true;
    }
    return false;
}
bool CCourseMgr::AddCourse(const string &sCourseName)
{
    CCourseInfo tmpInfo(sCourseName);
    return this->AddCourse(tmpInfo);
}
bool CCourseMgr::DelCourse(OBJID idCourse)
{
    COURSE_MAP mapDel;
    for (const auto &rInfo : m_mapCourse)
    {
        if (rInfo.first == idCourse)
        {
            mapDel.insert(rInfo);
        }
    }
    if (mapDel.size() > 0)
    {
        for (const auto &rDelInfo : mapDel)
        {
            auto iter_it = m_mapCourse.find(rDelInfo.first);
            if (iter_it != m_mapCourse.end())
            {
                m_mapCourse.erase(iter_it);
            }
        }
        return true;
    }
    return false;
}
void CCourseMgr::DelLatestCourse()
{
    auto erase_it = m_mapCourse.rbegin();
    m_mapCourse.erase(erase_it->first);
}
bool CCourseMgr::DelCourseByName(const string &sCourseName)
{
    try
    {
        COURSE_MAP mapDel;
        for (const auto &rInfo : m_mapCourse)
        {
            if (rInfo.second->GetCourseName() == sCourseName)
            {
                mapDel.insert(rInfo);
            }
        }
        if (mapDel.size() > 0)
        {
            for (const auto &rDelInfo : mapDel)
            {
                auto iter_it = m_mapCourse.find(rDelInfo.first);
                if (iter_it != m_mapCourse.end())
                {
                    m_mapCourse.erase(iter_it);
                }
            }
            return true;
        }
        return false;
    }
    catch (const exception& e)
    {
        cout << "file:" << __FILE__ << "line:" << __LINE__ << endl;
        cout << e.what() << endl;
        return false;
    }
}
void  CCourseMgr::PrintCourseList(stringstream &ss) const
{
    try
    {
        for (const auto &rInfo : m_mapCourse)
        {
            shared_ptr<CCourseInfo> pref = rInfo.second;
            pref->PrintInfo(ss);
            pref->AnnounceHomework(ss);
        }
    }
    catch (const exception& e)
    {
        cout << "file:" << __FILE__ << "line:" << __LINE__ << endl;
        cout << e.what() << endl;
    }
}
void CCourseMgr::FindLongNameCourse(stringstream &ss)
{
    try
    {
        typedef list<string/*sCourseName*/> listLonggestCourse;
        listLonggestCourse courseNamelist;
        auto map_it = m_mapCourse.cbegin();
        CCourseInfo info = *(map_it->second);
        string longestCourseName = info.GetCourseName();
        auto uMaxLen = longestCourseName.length();
        courseNamelist.push_back(longestCourseName);
        ++map_it;
        while (map_it != m_mapCourse.cend())
        {
            CCourseInfo tmpInfo = *(map_it->second);
            string tmpStr = tmpInfo.GetCourseName();
            if (tmpStr.length() > uMaxLen)
            {
                if (courseNamelist.size() > 0)
                {
                    courseNamelist.clear();
                }
                courseNamelist.push_back(tmpStr);
                uMaxLen = tmpStr.length();
            }
            else if (uMaxLen == tmpStr.length())
            {
                courseNamelist.push_back(tmpStr);
            }
            else
            {
                ;
            }
            ++map_it;
        }
        ss << "名字最长的课程是:" << endl;
        for (const auto &rCourseName : courseNamelist)
        {
            ss << rCourseName << endl;
        }
        auto list_it = courseNamelist.begin();
        //ss << "最长课程长度是：" << list_it->size() << endl;
        ss << "最长课程长度是:" << list_it->size() << endl;
    }
    catch (const exception& e)
    {
        cout << "file:" << __FILE__ << "line:" << __LINE__ << endl;
        cout << e.what() << endl;
    }
}
void CCourseMgr::SaveCourse(fstream &fs)
{
    try
    {
        if (!fs.is_open())
        {
            throw logic_error("file open err!");
        }
        fs.seekg(0, ios::end);//文件指针移到末尾
        for (const auto &rInfo : m_mapCourse)
        {
            fs << "courseId:" << 
            rInfo.second->GetCourseId() << " " 
            << "courseName:" << rInfo.second->GetCourseName() << endl;
        }
        fs.close();
    }
    catch (const exception& e)
    {
        cout << "file:" << __FILE__ << "line:" << __LINE__ << endl;
        cout << e.what() << endl;
    }
}
void CCourseMgr::SortCourse(sortEnum sortType, vector<string> &vecSort)
{
    try
    {
        for (const auto &rInfo : m_mapCourse)
        {
            string str = rInfo.second->GetCourseName();
            vecSort.push_back(str);
        }
        sort(vecSort.begin(), vecSort.end(), [sortType](string a, string b){
            if (sortType == sortFromLesstoMore)
            {
                return a < b;
            }
            else
            {
                return a > b;
            }
        });
    }
    catch (const exception& e)
    {
        cout << "file:" << __FILE__ << "line:" << __LINE__ << endl;
        cout << e.what() << endl;
    }
}
bool  CCourseMgr::SearchCourseByName(string szName, vector<string> &vecCorrect)
{
    try
    {
        bool bsucc = false;
        for (const auto &rInfo : m_mapCourse)
        {
            string str = rInfo.second->GetCourseName();
            if (str.find(szName) != std::string::npos)
            {
                vecCorrect.push_back(str);
                bsucc = true;
            }
        }
        return bsucc;
    }
    catch (const exception& e)
    {
        cout << "file:" << __FILE__ << "line:" << __LINE__ << endl;
        cout << e.what() << endl;
        return false;
    }
}
```
##cmd.h  

```C++
#ifndef  __CMD_H_
#define __CMD_H_
#include <map>
#include <string>
#include <memory>
#include <sstream>
#include <cctype>
#include "course.h"
#include "Cmd.h"
using namespace std;
const string scnLogPath = "cnResult.log";
const string senLogPath = "enResult.log";
typedef enum lanuageEnum
{
    simpleChinese        = 1,
    english                = 2,
};
enum exitCode
{
    code_exit = 0,//退出程序
    code_circle = 1,//继续循环
};
enum courseCmd
{
    Cmd_PrintHelpInfo = 1,
    Cmd_PrintCourseInfo = 2,
    Cmd_PrintCourseNum = 3,
    Cmd_PrintlongName = 4,
    Cmd_DeleteLastCourse = 5,
    Cmd_DelCourseById = 6,
    Cmd_DelCourseByName = 7,
    Cmd_sortFromLess = 8,
    Cmd_sortFromMore = 9,
    Cmd_SearchByName = 10,
    Cmd_TestIdOPerator = 11,
    Cmd_exitCourseMgr = 12,
};
class Clog
{
public:
    Clog() = default;
    ~Clog();
    bool init(const string &sFileName);
    void WriteLog(const stringstream &ss);
    void SaveLog();
private:
    ofstream m_outfs;
};
class Ccmd
{
public:
    Ccmd()=default;
    virtual ~Ccmd();
    bool init(const string &sFileName,shared_ptr<CCourseMgr> pMgrRef);
    exitCode inputCmd(string strCmd);
    void SetLanguageType(lanuageEnum languageType) { m_languageType = languageType; };
protected:
    void BindCmdToOper(string strCmd, courseCmd cmdOper);
    virtual void PrintHelpInfo();
    virtual void PrintCourseInfo();
    virtual void PrintCourseNum();
    virtual void PrintlongName();
    virtual void DeleteLastCourse();
    virtual bool DelCourseById(OBJID idCourse);
    virtual bool DelCourseByName(const string &sCourseName);
    virtual void SortCourse(sortEnum sortType);
    virtual void SearchCourseByName(string szName);
    virtual void TestIdOPerator(OBJID id);
    virtual void ExitMgr() = 0;
    virtual string OperSucc() = 0;
    virtual string OperFail() = 0;
private:
    void ShowStream();
    string InputCourseName();
    bool InputCourseId(OBJID &idCourse);
private:
    lanuageEnum m_languageType;
    typedef std::map<string, courseCmd> MAP_OPER;
    MAP_OPER m_mapCmd;
    Clog log;
    shared_ptr<CCourseMgr> m_mgrRef = nullptr;
protected:
    stringstream m_ss;
};
class CEnCcmd:public Ccmd//英文命令
{
public:
    CEnCcmd();
    ~CEnCcmd() = default;
private:
protected:
    virtual void PrintHelpInfo() override;
    virtual void PrintCourseInfo() override;
    virtual void PrintCourseNum() override;
    virtual void PrintlongName() override;
    virtual void DeleteLastCourse() override;
    virtual bool DelCourseById(OBJID idCourse) override;
    virtual bool DelCourseByName(const string &sCourseName) override;
    virtual void SortCourse(sortEnum sortType) override;
    virtual void SearchCourseByName(string szName) override;
    virtual void TestIdOPerator(OBJID id) override;
    virtual void ExitMgr() override;
    virtual string OperSucc() override;
    virtual string OperFail() override;
};
class CCnCcmd :public Ccmd//中文命令
{
public:
    CCnCcmd();
    ~CCnCcmd() = default;
private:
protected:
    virtual void PrintHelpInfo() override;
    virtual void PrintCourseInfo() override;
    virtual void PrintCourseNum() override;
    virtual void PrintlongName() override;
    virtual void DeleteLastCourse() override;
    virtual bool DelCourseById(OBJID idCourse) override;
    virtual bool DelCourseByName(const string &sCourseName) override;
    virtual void SortCourse(sortEnum sortType) override;
    virtual void SearchCourseByName(string szName) override;
    virtual void TestIdOPerator(OBJID id) override;
    virtual void ExitMgr() override;
    virtual string OperSucc() override;
    virtual string OperFail() override;
};
class CcmdMgr
{
public:
    bool CreateSys(lanuageEnum languageType, shared_ptr<CCourseMgr> pMgrRef);//简单工厂
    void Input();
private:
    typedef std::map<lanuageEnum, shared_ptr<Ccmd>> MAP_CMD;
    MAP_CMD m_mapCmd;
};
#endif
```

##cmd.cpp  
```C++
#include "Cmd.h"
Clog::~Clog()
{
    if (m_outfs.is_open())
    {
        m_outfs.close();
    }
}
bool Clog::init(const string &sFileName)
{
    try
    {
        m_outfs.open(sFileName, ios::out);
        if (!m_outfs.is_open())
        {
            throw logic_error("file open err!");
            return false;
        }
        return true;
    }
    catch (const exception& e)
    {
        cout << "file:" << __FILE__ << "line:" << __LINE__ << endl;
        cout << e.what() << endl;
        return false;
    }
}
void Clog::WriteLog(const stringstream &ss)
{
    m_outfs << ss.str() << endl;
}
bool Ccmd::init(const string &sFileName, shared_ptr<CCourseMgr> pMgrRef)
{
    if (!m_mgrRef)
        m_mgrRef = pMgrRef;
    return log.init(sFileName);
}
Ccmd::~Ccmd()
{
    m_mapCmd.clear();
}
void Ccmd::BindCmdToOper(string strCmd, courseCmd cmdOper)
{
    m_mapCmd.insert(make_pair(strCmd, cmdOper));
}
exitCode Ccmd::inputCmd(string strCmd)
{
    auto iter = m_mapCmd.find(strCmd);
    if (iter != m_mapCmd.end())
    {
        courseCmd cmd = iter->second;
        switch (cmd)
        {
        case Cmd_PrintHelpInfo:
        {
            PrintHelpInfo();
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_PrintCourseInfo:
        {
            PrintCourseInfo();
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_PrintCourseNum:
        {
            PrintCourseNum();
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_PrintlongName:
        {
            PrintlongName();
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_DeleteLastCourse:
        {
            DeleteLastCourse();
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_DelCourseById:
        {
            OBJID id;
            if (InputCourseId(id))
            {
                DelCourseById(id);
            }
            else
            {
                m_ss << OperFail() << endl;;
            }
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_DelCourseByName:
        {
            string tip = OperFail();
            string szName = InputCourseName();
            DelCourseByName(szName);
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_sortFromLess:
        {
            Ccmd::SortCourse(sortFromLesstoMore);
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_sortFromMore:
        {
            Ccmd::SortCourse(sortFromMoretoLess);
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_SearchByName:
        {
            string szName = InputCourseName();
            SearchCourseByName(szName);
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_TestIdOPerator:
        {
            OBJID id;
            if (InputCourseId(id))
            {
                TestIdOPerator(id);
            }
            else
            {
                m_ss << OperFail() << endl;;
            }
            log.WriteLog(m_ss);
            ShowStream();
            break;
        }
        case Cmd_exitCourseMgr:
        {
            ExitMgr();
            log.WriteLog(m_ss);
            ShowStream();
            return code_exit;
            break;
        }
        default:
            break;
        }
    }
    return code_circle;
}
void Ccmd::PrintHelpInfo()
{
    switch (m_languageType)
    {
    case simpleChinese:
        m_ss << "Cmd 输入语言 :中文" << endl;
        break;
    case english:
        m_ss << "Cmd input language :english" << endl;
        break;
    default:
        m_ss << "Cmd input language :invalid" << endl;
        break;
    }
}
void Ccmd::PrintCourseInfo()
{
    m_mgrRef->PrintCourseList(m_ss);
}
void Ccmd::PrintCourseNum()
{
    m_ss << m_mgrRef->GetCourseNum() << endl;
}
void Ccmd::PrintlongName()
{
    m_mgrRef->FindLongNameCourse(m_ss);
}
void Ccmd::DeleteLastCourse()
{
    m_mgrRef->DelLatestCourse();
    m_ss<<OperSucc()<<endl;
    PrintCourseInfo();
}
bool Ccmd::DelCourseById(OBJID idCourse)
{
    if (m_mgrRef->DelCourse(idCourse))
    {
        m_ss << OperSucc() << endl;
        PrintCourseInfo();
        return true;
    }
    m_ss << OperFail() << endl;
    return false;
}
bool Ccmd::DelCourseByName(const string &sCourseName)
{
    if (m_mgrRef->DelCourseByName(sCourseName))
    {
        m_ss << OperSucc() << endl;
        PrintCourseInfo();
        return true;
    }
    m_ss << OperFail() << endl;
    return false;
}
void Ccmd::SortCourse(sortEnum sortType)
{
    vector<string> vecSort;
    m_mgrRef->SortCourse(sortType, vecSort);
    for (const auto &r : vecSort)
    {
        m_ss << r << " ";
    }
    m_ss << endl;
}
void Ccmd::SearchCourseByName(string szName)
{
    vector<string> vecStr;
    if (m_mgrRef->SearchCourseByName(szName, vecStr))
    {
        for (const auto &r : vecStr)
        {
            m_ss << r << " ";
        }
        m_ss << endl;
    }
    else
    {
        m_ss << OperFail() << endl;
    }
}
void Ccmd::TestIdOPerator(OBJID id)
{
    shared_ptr<CCourseInfo> pRef = m_mgrRef->operator[] (id);
    m_ss << *pRef << endl;
}
void Ccmd::ShowStream()
{
    //std::cout << m_ss.rubf() << endl;
    string s = m_ss.str();
    cout<<s<<endl;
    m_ss.str("");
}
string Ccmd::InputCourseName()
{
    string sCourseName;
    cin >> sCourseName;
    return sCourseName;
}
bool Ccmd::InputCourseId(OBJID &idCourse)
{
    char input;
    cin >> input;
    if (!isdigit(input))
    {
        m_ss << OperFail() << endl;
        log.WriteLog(m_ss);
        ShowStream();
        return false;
    }
    else
    {
        idCourse = input - '0';
        return true;
    }
}
CEnCcmd::CEnCcmd()
{
    BindCmdToOper("one", Cmd_PrintHelpInfo);
    BindCmdToOper("two", Cmd_PrintCourseInfo);
    BindCmdToOper("three", Cmd_PrintCourseNum);
    BindCmdToOper("four", Cmd_PrintlongName);
    BindCmdToOper("five", Cmd_DeleteLastCourse);
    BindCmdToOper("six", Cmd_DelCourseById);
    BindCmdToOper("seven", Cmd_DelCourseByName);
    BindCmdToOper("eight", Cmd_sortFromLess);
    BindCmdToOper("nine", Cmd_sortFromMore);
    BindCmdToOper("ten", Cmd_SearchByName);
    BindCmdToOper("elevent", Cmd_TestIdOPerator);
    BindCmdToOper("twelve", Cmd_exitCourseMgr);
}
void CEnCcmd::PrintHelpInfo()
{
    m_ss << "shiyanlou manager system help" << endl;
    m_ss << "input:mean" << endl;
    m_ss << "one:printHelpInfo" << endl;
    m_ss << "two:printCourseInfo" << endl;
    m_ss << "three:rintCourseNum" << endl;
    m_ss << "four:printlonggestName" << endl;
    m_ss << "five:eleteLastCourse" << endl;
    m_ss << "six:DelCourseById" << endl;
    m_ss << "seven:DelCourseByName" << endl;
    m_ss << "eight:SortCourse->sortFromLesstoMore" << endl;
    m_ss << "nine:SortCourse->sortFromMoretoLess" << endl;
    m_ss << "ten:SearchCourseByName" << endl;
    m_ss << "elevent:estIdOPerator" << endl;
    m_ss << "twelve:Exit" << endl;
}
void CEnCcmd::PrintCourseInfo()
{
    m_ss << "PrintCourseInfo:" << endl;
    Ccmd::PrintCourseInfo();
}
void CEnCcmd::PrintCourseNum()
{
    m_ss << "PrintCourseNum:" << endl;
    Ccmd::PrintCourseInfo();
}
void CEnCcmd::PrintlongName()
{
    m_ss << "PrintlongName:" << endl;
    Ccmd::PrintCourseInfo();
}
void CEnCcmd::DeleteLastCourse()
{
    m_ss << "DeleteLastCourse:" << endl;
    Ccmd::DeleteLastCourse();
}
bool CEnCcmd::DelCourseById(OBJID idCourse)
{
    m_ss << "DelCourseById:" << idCourse<< endl;
    return Ccmd::DelCourseById(idCourse);
}
bool CEnCcmd::DelCourseByName(const string &sCourseName)
{
    m_ss << "DelCourseByName:" << sCourseName << endl;
    return Ccmd::DelCourseByName(sCourseName);
}
void CEnCcmd::SortCourse(sortEnum sortType)
{
    m_ss << "SortCourse:" << sortType << endl;
    Ccmd::SortCourse(sortType);
}
void CEnCcmd::SearchCourseByName(string szName)
{
    m_ss << "SearchCourseByName:" << szName << endl;
    Ccmd::SearchCourseByName(szName);
}
void CEnCcmd::TestIdOPerator(OBJID id)
{
    m_ss << "TestIdOPerator:" << id << endl;
    Ccmd::TestIdOPerator(id);
}
void CEnCcmd::ExitMgr()
{
    m_ss << "ExitMgr" << endl;
}
string CEnCcmd::OperSucc()
{
    string szSucc = "operSucc";
    return szSucc;
}
string CEnCcmd::OperFail()
{
    string szFail = "operFail";
    return szFail;
}
CCnCcmd::CCnCcmd()
{
    BindCmdToOper("1", Cmd_PrintHelpInfo);
    BindCmdToOper("2", Cmd_PrintCourseInfo);
    BindCmdToOper("3", Cmd_PrintCourseNum);
    BindCmdToOper("4", Cmd_PrintlongName);
    BindCmdToOper("5", Cmd_DeleteLastCourse);
    BindCmdToOper("6", Cmd_DelCourseById);
    BindCmdToOper("7", Cmd_DelCourseByName);
    BindCmdToOper("8", Cmd_sortFromLess);
    BindCmdToOper("9", Cmd_sortFromMore);
    BindCmdToOper("10", Cmd_SearchByName);
    BindCmdToOper("11", Cmd_TestIdOPerator);
    BindCmdToOper("12", Cmd_exitCourseMgr);
}
void CCnCcmd::PrintHelpInfo()
{
    m_ss << "实验楼课程管理程序帮助列表" << endl;
    m_ss << "输入命令：含义" << endl;
    m_ss << "1：打印出程序帮助信息" << endl;
    m_ss << "2：打印程序存贮所有课程id和名字" << endl;
    m_ss << "3：打印课程数量" << endl;
    m_ss << "4：打印名字最长的课程，可有多个" << endl;
    m_ss << "5：删除最后一个课程" << endl;
    m_ss << "6:输入课程id删除课程" << endl;
    m_ss << "7:输入课程名删除课程" << endl;
    m_ss << "8:从小到大排序" << endl;
    m_ss << "9:从大到小排序" << endl;
    m_ss << "10:按文件名模糊查找课程" << endl;
    m_ss << "11：下标运算符测试" << endl;
    m_ss << "12:退出程序" << endl;
}
void CCnCcmd::PrintCourseInfo()
{
    m_ss << "输入命令:打印所有课程信息" << endl;
    Ccmd::PrintCourseInfo();
}
void CCnCcmd::PrintCourseNum()
{
    m_ss << "输入命令:打印课程数量" << endl;
    Ccmd::PrintCourseInfo();
}
void CCnCcmd::PrintlongName()
{
    m_ss << "输入命令:打印名字最长的课程" << endl;
    Ccmd::PrintCourseInfo();
}
void CCnCcmd::DeleteLastCourse()
{
    m_ss << "输入命令:删除最新一门课程" << endl;
    Ccmd::DeleteLastCourse();
}
bool CCnCcmd::DelCourseById(OBJID idCourse)
{
    m_ss << "输入命令:删除课程,id:" << idCourse << endl;
    return Ccmd::DelCourseById(idCourse);
}
bool CCnCcmd::DelCourseByName(const string &sCourseName)
{
    m_ss << "输入命令:删除课程,name:" << sCourseName << endl;
    return Ccmd::DelCourseByName(sCourseName);
}
void CCnCcmd::SortCourse(sortEnum sortType)
{
    m_ss << "显示排序后信息,type:" <<sortType<< endl;
    Ccmd::SortCourse(sortType);
}
void CCnCcmd::SearchCourseByName(string szName)
{
    m_ss << "按课程名查找结果：" << endl;
    Ccmd::SearchCourseByName(szName);
}
void CCnCcmd::TestIdOPerator(OBJID id)
{
    m_ss << "下标运算符测试结果为:" << endl;
    Ccmd::TestIdOPerator(id);
}
void CCnCcmd::ExitMgr()
{
    m_ss << "退出管理系统"<<endl;
}
string CCnCcmd::OperSucc()
{
    string szSucc = "操作成功";
    return szSucc;
}
string CCnCcmd::OperFail()
{
    string szFail = "操作失败";
    return szFail;
}
bool CcmdMgr::CreateSys(lanuageEnum languageType, shared_ptr<CCourseMgr> pMgrRef)
{
    try
    {
        shared_ptr<Ccmd> pCmdRef = nullptr;
        switch (languageType)
        {
        case simpleChinese:
        {
            pCmdRef = make_shared<CCnCcmd>();
            pCmdRef->init(scnLogPath, pMgrRef);
            pCmdRef->SetLanguageType(languageType);
        }
        break;
        case english:
        {
            pCmdRef = make_shared<CEnCcmd>();
            pCmdRef->init(senLogPath, pMgrRef);
            pCmdRef->SetLanguageType(languageType);
        }
        break;
        default:
            break;
        }
        if (pCmdRef != nullptr)
        {
            m_mapCmd.insert(make_pair(languageType, pCmdRef));
        }
    }
    catch (const exception& e)
    {
        cout << "file:" << __FILE__ << "line:" << __LINE__ << endl;
        cout << e.what() << endl;
        return false;
    }
}
void CcmdMgr::Input()
{
    int szNum;
    exitCode code;
    cout << "choose language:1:chinese;2:english;"<<endl;
    cin >> szNum;
    switch (szNum)
    {
    case 1:
    {
        auto iter = m_mapCmd.find(simpleChinese);
        if (iter != m_mapCmd.end())
        {
            shared_ptr<Ccmd> pCmdRef = iter->second;
            cout<<"中文操作系统,请输入命令，输入1打印命令帮助"<<endl;
            while(1)
            {
                string szInput;
                cin>>szInput;
                code = pCmdRef->inputCmd(szInput);
                if(code_exit == code )
                {
                    return;
                }
            }
        }
    }
    case 2:
    {
        auto iter = m_mapCmd.find(english);
        if (iter != m_mapCmd.end())
        {
            shared_ptr<Ccmd> pCmdRef = iter->second;
            cout<<"english operation system,
            please input cmd,input one to PrintHelp"<<endl;
            while(1)
            {
                string szInput;
                cin>>szInput;
                code = pCmdRef->inputCmd(szInput);
                if(code_exit == code )
                {
                    return;
                }
            }
        }
    }
    default:
        cout<<"invalid operation,exit system"<<endl;
        break;
    }
}
```
##main.cpp
```C++
#include "course.h"
#include <sstream>
#include <memory>
#include <string>
#include "Cmd.h"
using namespace std;
struct courseData
{
    courseTypeEnum type;
    string courseName;
    string strHomeWork;
};
int main()
{
    std::vector<courseData> courseVec;
    courseData data1={xm_course,"C++","每周天晚上20:00交作业"};
    courseVec.push_back(data1);
    courseData data2={pg_course,"HTML","时间剩余60分钟"};
     courseVec.push_back(data2);
    courseData data3={pg_course,"HTML5","时间剩余120分钟"};
     courseVec.push_back(data3);
    courseData data4={xm_course,"NodeJS","下周一晚上20:00交作业"};
     courseVec.push_back(data4);
    courseData data5={xm_course,"Shell","每周六晚上20:00交作业"};
     courseVec.push_back(data5);
    courseData data6={xm_course,"Python","即将开始"};
     courseVec.push_back(data6);
    courseData data7={pg_course,"Linux","时间剩余180分钟"};
     courseVec.push_back(data7);
    shared_ptr<CCourseMgr> pcourseMgrRef = make_shared<CCourseMgr>();
    for(const auto&r:courseVec)
    {
       pcourseMgrRef->CreateCourse(r.type,r.courseName,r.strHomeWork);
    }
    shared_ptr<CcmdMgr> pCmdMgrRef = make_shared<CcmdMgr>();
    pCmdMgrRef->CreateSys(simpleChinese,pcourseMgrRef);
    pCmdMgrRef->CreateSys(english,pcourseMgrRef);
    pCmdMgrRef->Input();
    return 0;
}
```


