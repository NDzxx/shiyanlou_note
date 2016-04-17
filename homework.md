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
    return (this->m_idCourse == rInfo.m_idCourse) && (this->m_sCouseName == rInfo.m_sCouseName);
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
    os << "courseId:os" << rInfo.m_idCourse << "+" << "courseName:" << rInfo.m_sCouseName << endl;
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
bool  CCourseMgr::CreateCourse(courseTypeEnum type, string strCourseName, string strHomeWork)
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
    COURSE_MAP tmpMap = rCourseMgr.m_mapCourse;//转1手，防止调用的是this,丢失了vec内的内容
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
            fs << "courseId:" << rInfo.second->GetCourseId() << " " << "courseName:" << rInfo.second->GetCourseName() << endl;
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

