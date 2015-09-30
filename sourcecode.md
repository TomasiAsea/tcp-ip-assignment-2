Source Code
Client.c

#include "inet.h"
#include "queue.h"
#include "qoperate.h"
#include <string.h>
#define BUFSIZE 1024

main(int argc, char *argv[]){
	int sockfd;
	int filefd, nbyte;
	int pID, pID1;
	int i, j, x, y;
	char buffer[BUFSIZE+1], choice;
	struct sockaddr_in serv_addr;
	int pipefd[2];
	char *msg = "Virtual Shop", *msgU = "TrueLogin", *msgA = "LogAdmin";
	char str[20];
	char pass[20] = "admin";
	char User[20];
	char admin[20];
	char flag = 'T';
	

	


	if(argc <= 1){
		printf("How to use : %s remoteIPaddress [example: ./client 127.0.0.1]\n", argv[0]);exit(1);}

	bzero( (char *)&serv_addr, sizeof(serv_addr) ); //
	serv_addr.sin_family = AF_INET;
	serv_addr.sin_port = htons (SERV_TCP_PORT);
	inet_pton (AF_INET, argv[1], &serv_addr.sin_addr);

	if( (sockfd = socket(AF_INET, SOCK_STREAM, 0) ) < 0) {
		perror("Client: socket() error\n");
		exit(1);}

	if(connect(sockfd,(struct sockaddr *)&serv_addr,sizeof(serv_addr))<0){
		perror("Client: connect() error\n");
		exit(1);}
			
		if ( pipe(pipefd) ==-1) { //if the pipe is not created, then it is a pipe error
			perror("pipe error");
			exit(1); }

		write (pipefd[1], msg, BUFSIZE);//writes msg1 from above into the pipe
		read(pipefd[0], buffer, BUFSIZE);//retrieves msg1 from pipe and place it into buffer
		send(sockfd,buffer,BUFSIZE,0);
		printf("%s\n", buffer);
		bzero(buffer, sizeof(buffer));

		
		do{
		while(flag == 'T'){
		printf("\n#################################\n");
		printf("Virtual     Shop \n");
		printf("1 to Register\n");
		printf("2 to Login \n");
		printf("3 Admin Access\n");
		
		
		static struct sigaction act;
		act.sa_handler = SIG_IGN;
		sigfillset (&(act.sa_mask));
		sigaction (SIGINT, &act, (void *)0);	
		
		scanf("%s", &choice);

		switch(choice){
			case '1':
				filefd = open("register.txt", O_WRONLY|O_APPEND, 0755);
				write(1, "User Name- Not more than 20",27);
				nbyte = read(0,buffer, 20);
				wait();
				write(filefd, buffer, nbyte);
				close(filefd);

				filefd = open("register.txt", O_RDONLY);
				read(filefd, buffer, 100);
				printf("%s",buffer);
				bzero(buffer,sizeof(buffer));
				close(filefd);

				
				break;

			case '2':
				
				
				write(1, "User Name : ",12);
				memset(&buffer, 0, sizeof(buffer));
				scanf("%s", User);
				

				FILE *fp = fopen("register.txt", "r");
				rewind(fp);
				
				while(fscanf(fp, "%s", str) != EOF){
					i = strlen(str);
					j = strlen(User);
					if(i == j){
						printf("Match\n");
						if ( pipe(pipefd) ==-1) { //if the pipe is not created, //then it is a pipe error
							perror("pipe error");
							exit(1); }
						memset(&buffer, 0, sizeof(buffer));
						write (pipefd[1], msgU, BUFSIZE);//writes msg1 from above //into the pipe
						read(pipefd[0], buffer, BUFSIZE);//retrieves msg1 from //pipe and place it into buffer
						printf("%s\n", buffer);
						pID1 = fork();
						if(pID1 == 0){
							execl("user", "user", (char*)0);
						}
						else{
							wait((int *)0);
							printf("User Served\n");}
						flag = 'F';
						break;}
					else
						printf("Nope\n");
				

				}
				fclose(fp);
				
				
				
				break;

			case '3':
				memset(&buffer, 0, sizeof(buffer));
				write(1, "Admin Name : ",13);
				scanf("%s", admin);

				x = strlen(pass);
				y = strlen(admin);
				if(x == y){
					printf("logged \n");
					if ( pipe(pipefd) ==-1) { //if the pipe is not created, then it //is a pipe error
							perror("pipe error");
							exit(1); }
					bzero(buffer,sizeof(buffer));
					write (pipefd[1], msgA, BUFSIZE);//writes msg1 from above into //the pipe
					read(pipefd[0], buffer, BUFSIZE);//retrieves msg1 from pipe and //place it into buffer
					printf("%s\n", buffer);
					pID = fork();
						if(pID == 0){
							execl("admin", "admin", (char*)0);
							//execl("/bin/ls", "ls", "-al", (char *)0);
							}
						else{
							wait((int *)0);
							printf("Admin Served\n");}
					flag = 'F';}
					else{
						printf("wrong\n");}
				
				
				break;

			}
			
	
			
	}
	send(sockfd,buffer,BUFSIZE,0);
	flag = 'T';
	}while (strcmp(buffer,"/q"));

close(sockfd);	
}







user.c

#include <stdio.h>
#include <stdlib.h>
#include "queue.h"
#include "qoperate.h"

int main(){
	char flag = 'T';
	char choice;
	pid_t pid;

	while(flag == 'T'){
		printf("1 View Products :\n");
		printf("2 Purchase Products:\n");
		printf("3 Exit:\n");

		scanf("%s", &choice);

		switch (choice){
			case '1':
				switch (pid = fork()){
					case 0:
						serve();
						break;
					case -1:
						warn("Fork error");
						break;
					default:
						printf("PID for parent is %d\n", pid);
				}
				exit (pid != -1 ? 0 : 1);
				break;
			case 2:
				break;
			case 3:
				flag = 'F';
				break;
		}


	}
}






admin.c

#include <stdio.h>
#include <stdlib.h>
#include "queue.h"
#include "qoperate.h"

int main(){
	int priority = 1;
	char product[20];
	char flag = 'T';
	FILE *fp;
	char choice;
	char str[20];
	while(flag == 'T'){
		printf("1 add Products :\n");
		printf("2 View All users:\n");
		printf("3 Exit:\n");

		scanf("%s", &choice);

		switch (choice){
			case '1':
				if(priority == 10){
					priority = 1;
				}
				
				printf("Product Name : ");
				scanf("%s", product);
				if(enter (product, priority) < 0){
					warn("enter fails");
					exit(3);}
				priority += 1; 
				break;
			case 2:
				fp = fopen("register.txt", "r");
				rewind(fp);
				
				while(fscanf(fp, "%s", str) != EOF){
					puts(str);
				}
				break;
			case 3:
				flag = 'F';
				break;
		}
	}
}


qoperate.h
int enter (char *objname, int priority){
int len, s_qid;
struct q_entry s_entry;

if((len = strlen(objname)) > MAXOBN){ //check object length
warn("invalid priority level");
return (-1);}

if(priority > MAXPRIOR || priority <0){
warn("invalid priority level");
return (-1);}

if((s_qid = init_queue()) == -1) //check if queue ID match
return(-1);

s_entry.mtype = (long) priority;
strncpy (s_entry.mtext, objname, MAXOBN);

if(msgsnd (s_qid, &s_entry, len, 0) == -1){ //send message
perror("Msgsnd fails");
return (-1);}
else 
return (0);
}

int warn(char *s){
fprintf(stderr, "Warning:%s\n",s);}

 init_queue(void){
int queue_id;

if((queue_id = msgget(ftok("\temp",9), IPC_CREAT| QPERM)) == -1)//pass pathname and ID
perror("Msgget fails");
return(queue_id);}

int serve(void){
int mlen, r_qid;
struct q_entry r_entry;

if( (r_qid = init_queue()) == -1) //check if queue ID match
return (-1);
	
for(;;)
{
if((mlen = msgrcv(r_qid, &r_entry,MAXOBN, (-1* MAXPRIOR* 0),MSG_NOERROR))==-1){
perror("msgrcv faild");
return(-1);}
else{
r_entry.mtext[mlen] = '\0';
proc_obj (&r_entry);} }}

int proc_obj(struct q_entry *msg){
printf("\n Priority: %ld message %s\n", msg->mtype, msg->mtext);
}

queue.h

#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <string.h>
#include <errno.h>
#define Qkey ((key_t)0105) //ipc key for queue
#define QPERM 0660 //access permission
#define MAXOBN 50 //maximum length for object
#define MAXPRIOR 10 //maximum priority value

struct q_entry {//message structure for message queue
long mtype; //type of message
char mtext[MAXOBN+1]; //message
};

inet.h

#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <signal.h>
#include <fcntl.h>
#include <unistd.h>
#include <setjmp.h>

#define SERV_TCP_PORT 25000
#define SERV_UDP_PORT 35001
#define CLI_UDP_PORT 35002

Server.c

#include "inet.h"
#include <signal.h>
#define BUFSIZE 1024

int main(int argc, char** argv){
fd_set master;
fd_set read_fds;
struct sockaddr_in myaddr;
struct sockaddr_in remoteaddr;
int fdmax;
int sockfd;
int new_sockfd;
char buffer[BUFSIZE];
int nbytes;
int yes=1;
int addrlen;
int i,j;

sigset_t set1;
sigemptyset(&set1);
sigaddset(&set1, SIGTSTP);
sigprocmask(SIG_BLOCK, &set1, NULL);

FD_ZERO(&master); //clear the thing in master
FD_ZERO(&read_fds);//clear the thing in read_fds

if((sockfd = socket(AF_INET, SOCK_STREAM, 0))==-1) {
		printf("\nsocket() error!!!\n");
		exit(1);}

if(setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(int))==-1)  {
		printf("\nSetsockopt() error!!!\n");
		exit(1);}

bzero((char *)&myaddr, sizeof(myaddr));
myaddr.sin_family = AF_INET;
myaddr.sin_addr.s_addr = INADDR_ANY;
myaddr.sin_port = htons(SERV_TCP_PORT);
bzero(&(myaddr.sin_zero), 8);

if( bind(sockfd, (struct sockaddr *)&myaddr, sizeof(myaddr))==-1){
		printf("\nbind() error!!!\n");
		exit(1);}

if(listen(sockfd, 10) == -1){
		printf("\nlisten() error!!!\n");
		exit(1);}

FD_SET(sockfd, &master); //add sockfd into master
fdmax = sockfd; //only sockfd at this moment

for(;;){
	read_fds = master;
	if(pselect(fdmax+1, &read_fds, NULL, NULL,NULL, &set1) == -1){ //to prevent the signal race condition
		printf("select() error\n");
		exit(1);	}
	for(i=0; i<=fdmax; i++) {
		if( FD_ISSET(i,&read_fds)){
			if(i == sockfd) {
				addrlen = sizeof(remoteaddr);
				if((new_sockfd = accept(sockfd, &remoteaddr, &addrlen)) == -1) {
					printf("\naccept() error!!!\n"); }
				else {
					FD_SET(new_sockfd, &master); //add new_sockfd into master
					if(new_sockfd > fdmax) {
						fdmax = new_sockfd;}
				printf("selectserver: new connection from %s on socket %d \n",
						inet_ntoa(remoteaddr.sin_addr), new_sockfd);
				} //else for accept()
			}//else for i == sockfd
			else{ /*if i != sockfd*/
				if((nbytes = recv(i, buffer, sizeof(buffer), 0))<=0){
					if(nbytes == 0) {
						printf("selectserver: socket %d hung up\n", i);}
				else { 
					printf("\nrecv() error!!!\n"); }//else for nbytes ==0
				close(i);
				FD_CLR(i, &master);//remove i from master
			}//for recv
			else {
				printf("Message [%s] from soket [%d] client [%s] \n", buffer, i,inet_ntoa(remoteaddr.sin_addr));



				/*
				for(j=0; j<= fdmax; j++){
					if(FD_ISSET(j, &master)) {//test to check whether j is in master or not
							if(j==new_sockfd) {
								if(send(j, buffer, nbytes, 0)==-1){
								perror("\nsend() error!!!\n");}
								else
							printf("\nSend[%s] via soket [%d]\n",buffer,j);
						}// for j!=sockfd
					}}}*/}}}}}
return 0;
}
