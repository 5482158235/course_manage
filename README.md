#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<stdbool.h>

#define MAX 15

typedef struct Course
{

	int id;									//课程编号
	char name[MAX];							//课程名称
	double crs;								//学分 
	char nature[MAX];						//课程性质
	char term[MAX];							//开课学期 
	double All_hours;						//总学时
	double Teaching_hours;					//授课学时
	double Experimental_hours;				//实验学时
	
	int sort;								//用于排序标记 
	
	struct Course* next;
	
}Course;

typedef struct 
{
	
	struct Course* head;
	int size;
	
}List;

void main_menu();

void add_List(List* list);									
void add_Course(Course* course, List* list, int* b);

void scan_List(List* list);

void choose_menu();
void find_menu();
void find_Course(List* list); 
void find_Course_crs(List* list);
void find_Course_nature(List* list);

void delete_Course(List* list);
void delete_menu_1();
void delete_menu_2();
void delete_Course_id(List* list);
void delete_Course_name(List* list);

void sort_Course(List* list);

int load_Course(List* list);
void delete_file_name(List* list, char* name, bool* find);
void delete_file_id(List* list, int id, bool* find);

void save_Course(List* list, Course* temp);
void sort_file_Course(List* list);

void init_List(List* list)				//链表初始化 
{
	list->head = NULL;
	list->size = 0;
}

void save_Course(List* list, Course* temp) 		//将输入的数据存入文件 
{
    FILE* manage_Course = fopen("课程管理系统.txt", "a");
    
    if (manage_Course == NULL) 
	{
        perror("课程管理系统");
        return;
    }
    
    fprintf(manage_Course, "%d\t%s\t%.2f\t%s\t%s\t%.2f\t%.2f\t%.2f\n", 
			temp->id, temp->name, temp->crs, temp->nature,
            temp->term, temp->All_hours, temp->Teaching_hours, temp->Experimental_hours);
			
	fclose(manage_Course);
}

void delete_file_id(List* list, int id, bool* find)		//通过id删除文件夹中的数据 
{
    FILE* new_file = fopen("临时文件.txt", "w");
	if (new_file == NULL) {
	    perror("临时文件");
		return;
    }

    FILE* old_file = fopen("课程管理系统.txt", "r");
    if (old_file == NULL) {
        perror("课程管理系统");
        fclose(new_file);
        return;
    }
	
    Course* current = list->head;
    Course* prev = NULL; 				// 用于保存前一个节点的指针

    // 遍历链表和文件，删除匹配的课程
    int count = 0;
    int i = 0;
    for(i = 0; i < list->size; i++)
	{
        if (current->id == id) 
		{
			*find = true;
            // 从链表中删除当前节点
            if (prev == NULL) 
			{
                // 如果是头节点
                if(list->size == 1){
                	list->head = NULL;
				} 
				else{
                	list->head = current->next;
                }
            } 
			else 
			{
                // 如果不是头节点
                prev->next = current->next;
            }
            count++; // 记录删除的个数，等循环结束后再list->size -= count，否则会影响循环 

            // 释放当前节点的内存
            Course* temp = current;
            current = current->next;
            free(temp);
        } 
		else 
		{
            // 将当前节点的数据写入新文件
            fprintf(new_file, "%d\t%s\t%.2f\t%s\t%s\t%.2f\t%.2f\t%.2f\n",
                    current->id, current->name, current->crs, current->nature, current->term,
                    current->All_hours, current->Teaching_hours, current->Experimental_hours);

            // 移动到下一个节点
            prev = current;
            current = current->next;
        }
    }
    list->size -= count;
    fclose(old_file);
    fclose(new_file);
    
    // 删除原始文件并重命名临时文件
    int ret = remove("课程管理系统.txt");
    if(ret == 0){
    	printf("\n文件删除成功\n");
	}
	else{
		printf("\n文件删除失败\n");
		perror("");
		printf("\nerrno: %d\n", errno);
	}
    int rot = rename("临时文件.txt", "课程管理系统.txt");
    if(rot == 0){
    	printf("\n文件重命名成功\n");
	}
	else{
		printf("\n文件重命名失败\n");
	}
}

void delete_file_name(List* list, char* name, bool* find)		//通过名称删除文件夹中的数据 
{
	FILE* new_file = fopen("临时文件.txt", "a");
	
	if(new_file == NULL)
	{
		perror("临时文件");
		return;
	}
	
	FILE* old_file = fopen("课程管理系统.txt", "r");
	
	if(old_file == NULL)
	{
		perror("课程管理系统");
		fclose(new_file);
		return;
	}
	
	Course* current = list->head;
    Course* prev = NULL; // 用于保存前一个节点的指针

    // 遍历链表和文件，删除匹配的课程
    int count = 0;
    int i = 0;
    for(i = 0; i < list->size; i++)
	{
        if (strcmp(current->name, name) == 0) 
		{
			*find = true;
            // 从链表中删除当前节点
            if (prev == NULL) 
			{
                if(list->size == 1){
                	list->head = NULL;
				}
				else{
                	list->head = current->next;
                }
            } 
			else 
			{
                // 如果不是头节点
                prev->next = current->next;
            }
            count++;

            // 释放当前节点的内存
            Course* temp = current;
            current = current->next;
            free(temp);
        } 
		else 
		{
            // 将当前节点的数据写入新文件
            fprintf(new_file, "%d\t%s\t%.2f\t%s\t%s\t%.2f\t%.2f\t%.2f\n",
                    current->id, current->name, current->crs, current->nature, current->term,
                    current->All_hours, current->Teaching_hours, current->Experimental_hours);

            // 移动到下一个节点
            prev = current;
            current = current->next;
        }
    }
	list->size -= count;
	
    fclose(old_file);
    fclose(new_file);

    // 删除原始文件并重命名临时文件
    int ret = remove("课程管理系统.txt");
    if(ret == 0){
    	printf("\n文件删除成功\n");
	}
	else{
		printf("\n文件删除失败\n");
	}
    int rot = rename("临时文件.txt", "课程管理系统.txt");
    if(rot == 0){
    	printf("\n文件重命名成功\n");
	}
	else{
		printf("\n文件重命名失败\n");
	}
    
}

int load_List(List* list) {						//加载数据 
    FILE* file = fopen("课程管理系统.txt", "r");
    if (file == NULL) {
    	fclose(file);
		file = fopen("课程管理系统.txt", "w");
		fclose(file); 
    }
    file = fopen("课程管理系统.txt", "r");
    if(file == NULL){
    	perror("无法打开文件");
    	return 0;
	}

    Course* current;
    Course* newCourse;
    list->size = 0; // 重置链表大小

    while (!feof(file)) {
        // 为新课程节点分配内存
        newCourse = (Course*)malloc(sizeof(Course));
        if (newCourse == NULL) {
            perror("内存分配失败");
            break;
        }

        // 将文件中的课程信息读取到新课程节点中
        if (fscanf(file, "%d\t%s\t%lf\t%s\t%s\t%lf\t%lf\t%lf\n",
                    &newCourse->id, newCourse->name, &newCourse->crs,
                    newCourse->nature, newCourse->term,
                    &newCourse->All_hours, &newCourse->Teaching_hours, &newCourse->Experimental_hours) == 8) {
            // 将新节点添加到链表末尾
            newCourse->next = NULL;
            if (list->head == NULL) {
                list->head = newCourse;
            } 
			else {
                current = list->head;
                while (current->next != NULL) {
                    current = current->next;
                }
                current->next = newCourse;
            }
            list->size++; // 链表大小增加
        } 
		else {
            // 读取失败
            free(newCourse);
            return;
        }
    }

    fclose(file); // 关闭文件
    return 1;
}

void add_Course(Course* course, List* list, int* b)	 	//添加课程具体数据 
{
	
	printf("请输入课程编号： ");
	int input_result = scanf("%d", &course->id);
	
	if( input_result != 1)
	{
		printf("您输入的不是有效整数\n");
		*b = 0;
		return;
	}
	
	Course* current = list->head;
	int i = 0;
	for(i = 0; i < list->size; i++)
	{
		if(current->id == course->id)
		{
			*b = 0; 
			printf("\n该编号已存在，添加失败\n\n");
			system("pause");
			system("cls");
			return;
		}
		
		current = current->next;
	}
	
	printf("请输入课程名称： ");
	scanf("%s", &course->name);
	
	printf("请输入学分： ");
	scanf("%lf", &course->crs);
	
	printf("请输入课程性质： ");
	scanf("%s", &course->nature);
	
	printf("请输入开课学期： ");
	scanf("%s", &course->term);
	
	printf("请输入课程总学时： ");
	scanf("%lf", &course->All_hours);	
	
	printf("请输入授课学时： ");
	scanf("%lf", &course->Teaching_hours);
	
	printf("请输入实验学时： ");
	scanf("%lf", &course->Experimental_hours);
	
}

void add_List(List* list)				//课程录入 
{
	Course* newCourse = (Course*)malloc(sizeof(Course));
	
	if(newCourse == NULL)
	{
		printf("内存分配失败\n");
		return;
	}
	
	int a = 1;
	add_Course(newCourse, list, &a);
	
	if(a == 0)
	{
		free(newCourse);
		return;
	}
	
	if(list->head == NULL)
	{
		list->head = newCourse;
		list->size++;
		
		printf("添加成功\n");
		
		system("pause");
		system("cls");
		save_Course(list, newCourse);
		return;
	}
	
	Course* current = list->head; 
	
	int i = 0;
	for(i = 0; i < list->size - 1; i++)
	{
		current = current->next;
	}
	
	current->next = newCourse;
	
	list->size++;
	printf("添加成功\n");
	
	system("pause");
	system("cls");
	
	save_Course(list, newCourse);
	
	return;
	
}

void scan_List(List* list)				//课程浏览 
{
	printf("\n");
	
	if(list->size == 0){
		printf("暂无课程信息\n\n");
		
		system("pause");
		system("cls");
		return;
	}
	
	Course* current = list->head;

	int i = 0;
	for(i = 0; i < list->size; i++)
	{
		printf("%d\t\t", current->id);
		printf("%s\t\t", current->name);
		printf("%.1f\t\t", current->crs);
		printf("%s\t\t", current->nature);
		printf("%s\t\t", current->term);
		printf("%.1f\t\t", current->All_hours);
		printf("%.1f\t\t", current->Teaching_hours);
		printf("%.1f\n", current->Experimental_hours);
		
		current = current->next;
	}
	
	printf("\n");
	
	system("pause");
	system("cls");
	return;
	
}

void choose_menu()						//选择菜单 
{
	
	printf("************************\n");
	printf("****  是否继续查询  ****\n");
	printf("****     1、是      ****\n");
	printf("****     2、否      ****\n");
	printf("************************\n");
	
}

void find_Course_crs(List* list)		//按学分查询 
{

	int digit;
	printf("请输入您要查找的学分： ");
	double crs;
	scanf("%lf", &crs);
	
	Course* current = list->head;
	
	printf("\n");
	int i;
	for(i = 0; i < list->size; i++)
	{
		if(current->crs == crs)
		{
			
			printf("%d\t\t", current->id);
			printf("%s\t\t", current->name);
			printf("%.f\t\t", current->crs);
			printf("%s\t\t", current->nature);
			printf("%s\t\t", current->term);
			printf("%.2f\t\t", current->All_hours);
			printf("%.2f\t\t", current->Teaching_hours);
			printf("%.2f\n", current->Experimental_hours);
			
		}
		
		current = current->next;
	}
	printf("\n");
	
	system("pause");
	system("cls");
	
	choose_menu();
	scanf("%d", &digit);
	
	switch(digit)
	{
		case 1:
			find_Course(list);
			break;
			
		case 2:
			system("cls");
			return;
			
		default:
			system("cls");
			return;
	}
}

void find_Course_nature(List* list)		//按课程性质查询 
{
	
	char nature[MAX];
	int digit;
	
	printf("请输入您要查询的课程性质： ");
	scanf("%s", nature);
	
	Course* current = list->head;
	
	printf("\n");
	
	int i = 0;
	for(i = 0; i < list->size; i++)
	{
		if(strcmp(current->nature, nature) == 0)
		{
			
			printf("%d\t\t", current->id);
			printf("%s\t\t", current->name);
			printf("%.f\t\t", current->crs);
			printf("%s\t\t", current->nature);
			printf("%s\t\t", current->term);
			printf("%.2f\t\t", current->All_hours);
			printf("%.2f\t\t", current->Teaching_hours);
			printf("%.2f\n", current->Experimental_hours);
			
		}
		
		current = current->next;
	}
	printf("\n");
	
	system("pause");
	system("cls");
	
	choose_menu();
	scanf("%d", &digit);
	
	switch(digit)
	{
		case 1:
			find_Course(list);
			break;
			
		case 2:
			system("cls");
			return;
			
		default:
			system("cls");
			return;
	}
}

void find_menu()						//查询菜单 
{
	printf("*****************************\n");
	printf("****    1、按学分查询    ****\n");
	printf("****  2、按课程性质查询  ****\n");
	printf("****       0、退出       ****\n");
	printf("*****************************\n");
}

void find_Course(List* list)			//按学分或课程性质查询 
{
	if(list->size == 0)
	{
		printf("\n暂无课程信息\n\n");
		
		system("pause");
		system("cls");
		return;
	}
	
	system("cls");
	
	find_menu();
	int digit;
	
	scanf("%d", &digit);
	
	switch(digit)
	{
		case 1:
			find_Course_crs(list);
			break;
			
		case 2:
			find_Course_nature(list);
			break;
			
		case 0:
			system("cls");
			return;
			
		default:
			printf("请输入 0 ~ 1\n");
			system("pause");
			system("cls");
			
			find_Course(list); 
	}
}

void delete_menu_2()
{
	printf("************************\n");
	printf("****  是否继续删除  ****\n");
	printf("****     1、是      ****\n");
	printf("****     2、否      ****\n");
	printf("************************\n");
}

void delete_Course_id(List* list)			//按课程编号删除 
{
	int id;
	bool find = false;
	
	printf("请输入您要删除的课程编号: ");
	scanf("%d", &id);
	
	delete_file_id(list, id, &find);		//在文件中删除该数据 
	
	if(find){
		printf("\n删除成功\n\n");
	}
	else{
		printf("\n未查询到该id, 删除失败\n\n");
	}
	system("pause");
	system("cls");
	
	delete_menu_2();
	
	int digit;
	scanf("%d", &digit);
	
	switch(digit)
	{
		case 1:
			delete_Course(list);
			break;
			
		case 2:
			system("cls");
			return;
			
		default:
			system("cls");
			return;
	}
}

void delete_Course_name(List* list)			//按课程名称删除
{
	char name[MAX];
	bool find = false;
	
	printf("请输入您要删除的课程名称： ");
	scanf("%s", &name);
	
	delete_file_name(list, name, &find);			//在文件中删除该数据 

	if(find){
		printf("\n删除成功\n\n");
	}
	else{
		printf("\n未查询到该名称，删除失败\n\n");
	}
	system("pause");
	system("cls");
	
	delete_menu_2();
	
	int digit;
	scanf("%d", &digit);
	
	switch(digit)
	{
		case 1:
			delete_Course(list);
			break;
			
		case 2:
			system("cls");
			return;
			
		default:
			system("cls");
			return;
	}
}

void delete_menu_1()							//删除菜单 
{
	
	printf("*****************************\n");
	printf("****  1、按课程编号删除  ****\n");
	printf("****  2、按课程名称删除  ****\n");
	printf("****       0、退出       ****\n");
	printf("*****************************\n");
	
}

void delete_Course(List* list)				// 按照课程编号或课程名称 删除课程 
{
	if(list->size == 0)
	{
		printf("\n暂无课程信息\n\n");
		system("pause");
		system("cls");
		return; 
	}
	
	system("cls");
	
	delete_menu_1();
	
	int digit;
	scanf("%d", &digit);
	
	switch(digit)
	{
		case 1:
			delete_Course_id(list);
			
			break;
			
		case 2:
			delete_Course_name(list);
			break;
			
		case 0:
			system("cls");
			return;
			
		default:
			printf("请输入 0 ~ 2\n");
			system("pause");
			system("cls");
			
			delete_Course(list);
	}
}

void sort_Course(List* list) {
    if (list->size == 0) {
        printf("\n暂无课程信息\n\n");
        system("pause");
        system("cls");
        return;
    }
	
	int i = 0;
    Course* current = list->head;
    for(i = 0; i < list->size; i++) {
        int hasOne = strstr(current->term, "一") != NULL;
        int hasTwo = strstr(current->term, "二") != NULL;
        int hasThree = strstr(current->term, "三") != NULL;
        int hasFour = strstr(current->term, "四") != NULL;
        int isUpper = strstr(current->term, "上") != NULL;
        int isLower = strstr(current->term, "下") != NULL;

        // 根据包含的字符串设置 sort 值
        if (hasOne) {
            if(isUpper){
            	current->sort = 1;
			}
			else if(isLower){
				current->sort = 2;
			}
			else{
				current->sort = 10;
			}
        } 
		else if (hasTwo) {
            if(isUpper){
            	current->sort = 3;
			}
			else if(isLower){
				current->sort = 4;
			}
			else{
				current->sort = 10;
			}
        } 
		else if (hasThree) {
            if(isUpper){
            	current->sort = 5;
			}
			else if(isLower){
				current->sort = 6;
			}
			else{
				current->sort = 10;
			}
        } 
		else if (hasFour) {
            if(isUpper){
            	current->sort = 7;
			}
			else if(isLower){
				current->sort = 8;
			}
			else{
				current->sort = 10;
			}
        } 
		else {
            current->sort = 10; 
        }

        current = current->next;
    }

	int j = 0;
	for(j = 0; j < list->size - 1; j++){
        current = list->head;
        for(i = 0; i < list->size - 1; i++) {
            if (current->sort > current->next->sort) {
                // 交换两个节点的 sort 值				
                int tempSort = current->sort;
                char tempName[MAX]; 
                strcpy(tempName, current->name);
                double tempCrs = current->crs;
                char tempNature[MAX];
                strcpy(tempNature, current->nature);
                char tempTerm[MAX];
                strcpy(tempTerm, current->term);
                double tempAll_hours = current->All_hours;
                double tempTeaching_hours = current->Teaching_hours;
                double tempExperimental_hours = current->Experimental_hours;
                
                current->sort = current->next->sort;
                current->next->sort = tempSort;
                
                strcpy(current->name, current->next->name);
                strcpy(current->next->name, tempName);
                
                current->crs = current->next->crs;
                current->next->crs = tempCrs;
                
                strcpy(current->nature, current->next->nature);
                strcpy(current->next->nature, tempNature);
                
                strcpy(current->term, current->next->term);
                strcpy(current->next->term, tempTerm);
                
                current->All_hours = current->next->All_hours;
                current->next->All_hours = tempAll_hours;
                
                current->Teaching_hours = current->next->Teaching_hours;
                current->next->Teaching_hours = tempTeaching_hours;
                
                current->Experimental_hours = current->next->Experimental_hours;
                current->next->Experimental_hours = tempExperimental_hours;
                
            }
            current = current->next;
        }
    }
    
    sort_file_Course(list);
    
    printf("\n排序完成\n\n");
	system("pause");
	system("cls");
}


void sort_file_Course(List* list){			//对文件进行排序 
	FILE* manage_Course = fopen("临时文件.txt", "w");
	if(manage_Course == NULL){
		perror("临时文件");
		return;
	}
	Course* current = list->head;
	int i = 0;
	for(i = 0; i < list->size; i++){
		fprintf(manage_Course, "%d\t%s\t%.2f\t%s\t%s\t%.2f\t%.2f\t%.2f\n", 
			current->id, current->name, current->crs, current->nature,
            current->term, current->All_hours, current->Teaching_hours, current->Experimental_hours);
        
		current = current->next;    
	}
	
	fclose(manage_Course);
	remove("课程管理系统.txt");
	rename("临时文件.txt", "课程管理系统.txt");
}

void main_menu() 
{
	
	printf("***********************\n");
	printf("****  1、课程录入  ****\n");
	printf("****  2、课程浏览  ****\n");
	printf("****  3、课程查询  ****\n");
	printf("****  4、课程删除  ****\n");
	printf("****  5、课程排序  ****\n");
	printf("****    0、退出    ****\n");
	printf("***********************\n");
	
}

int main()
{
	
	List* list = ( List* ) malloc ( sizeof(List) );
	
	init_List(list);
	int load = load_List(list); 
	
	if(load == 0){
		return 0;
	}
	
	int digit;
	
	
	while(true)
	{
				
		main_menu();
		scanf("%d", &digit);
		
		switch(digit)
		{
			case 1:
				add_List(list);
				break;
				
			case 2:
				scan_List(list);
				break;
				
			case 3:
				find_Course(list);
				break;
				
			case 4:
				delete_Course(list);
				break;
				
			case 5:
				sort_Course(list); 
				break;
				
			case 0:
				printf("\n欢迎下次使用\n");
				return 0;
				
			default:
				printf("\n请输入 0 ~ 5\n\n");
				system("pause");
				system("cls");
				break;
		
		}
	}
}
