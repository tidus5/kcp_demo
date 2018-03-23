////////////////////////////////////////////////////////////////////////////
//// UDPServer.cpp
//https://www.cnblogs.com/findumars/p/5928712.html

#include <stdio.h>
#include <WINSOCK2.H>
#include <time.h>
#include <fcntl.h>
#include <strings.h>
#include <stdlib.h>

#include "ikcp.h"

#pragma comment(lib,"WS2_32.lib")
#define BUF_SIZE    128

SOCKET      socketSrv ;
SOCKADDR_IN addrSrv;
int hr;
SOCKADDR_IN addrClient;
char        buf[BUF_SIZE];

int udp_output(const char *buf, int len, ikcpcb *kcp, void *user)
{
    int res = sendto(socketSrv,buf, len,0,(SOCKADDR*)user,sizeof(addrClient));
    return 0;
}


int main(void)
{
    WSADATA wsd;
    SOCKET  s;
    int     nRet;

    // 初始化套接字动态库
    if(WSAStartup(MAKEWORD(2,2),&wsd) != 0)
    {
        printf("WSAStartup failed !\n");
        return 1;
    }

    // 创建套接字
    s = socket(AF_INET,SOCK_DGRAM,0);
    if(s == INVALID_SOCKET)
    {
        printf("socket() failed ,Error Code:%d\n",WSAGetLastError());
        WSACleanup();
        return 1;
    }

    socketSrv = socket(AF_INET,SOCK_DGRAM,0);
    int         len = sizeof(SOCKADDR);

    //设置为非阻塞模式
    int imode=1;
    nRet=ioctlsocket(socketSrv,FIONBIO,(u_long *)&imode);
    if(nRet == SOCKET_ERROR)
    {
        printf("ioctlsocket failed!");
        closesocket(socketSrv);
        WSACleanup();
        return -1;
    }

    // 设置服务器地址
    ZeroMemory(buf,BUF_SIZE);
    addrSrv.sin_addr.S_un.S_addr = htonl(INADDR_ANY);
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(5000);

    // 绑定套接字
    nRet = bind(socketSrv,(SOCKADDR*)&addrSrv,sizeof(SOCKADDR));
    if(SOCKET_ERROR == nRet)
    {
        printf("bind failed !\n");
        closesocket(s);
        WSACleanup();
        return -1;
    }

    // 从客户端接收数据
    printf(" server started..\n");
    ikcpcb *kcp = ikcp_create(1, (void*)&addrClient);
    kcp->output = udp_output;
    while(1)
    {
        Sleep(100);
        ikcp_update(kcp, clock());
        // 打印来自客户端发送来的数据
        while(1)
        {
            nRet = recvfrom(socketSrv,buf,BUF_SIZE,0,(SOCKADDR*)&addrClient,&len);
            if(nRet < 0)break;
            ikcp_input(kcp,buf,nRet);
        }

        while (1)
        {
            hr = ikcp_recv(kcp, buf, BUF_SIZE);
            // 没有收到包就退出
            if (hr < 0) break;
            // 如果收到包就回射
            printf("I recv :%s \n",buf);
            ikcp_send(kcp, buf, strlen(buf)+1);
        }
    }
    closesocket(s);
    WSACleanup();
    return 0;
}
