/* Program for Modeling the Shifting Cultvation Changes in Laung Prabhang province of Laos */

#include<stdio.h>
#include<stdlib.h>
#include<math.h>
#include<string.h>
#include<process.h> /*.........................................library for visual studio.....................................*/
#include"lp-global.h" /*................................ global delarations library..............................................*/

	short	d_road[LINE][PIXEL]; /*.................. Distance of grid from road ... for road buffer  ...........................................................*/
	int		HH_Inc_grid[3][LINE][PIXEL];  /*  NEED HH_Inc_grid[yr][row][col] --> yr=0,1,2 for as the Migration criteria */  /* what is 3 here ??? HH= HouseHold , Inc=Income , i.e addition of new grids ??  what is this array for ?*/ 
main()
{
	FILE	*fplu, *fpelev, *fpslope,  *fprivbuf , *fproad; /*  file pointers to open data files of bil format */
	FILE	*fppopln, *fpvil, *fpvilpt ; 
/*	FILE	*fpo,*fptmp2,*fptmp3,*fptmp4,*fp_inc, *fpvilptbuf; 	*/
	FILE	*fptmp /*used to read text data from txt files*/, *fp_inc, *fpcrp; 	
	
	unsigned char	*lubuf[B_SIZE], *elevbuf[B_SIZE], *slopebuf[B_SIZE], *rivbuffer[B_SIZE];
	unsigned char	*roadbuf[B_SIZE], *vilbuf[B_SIZE], *vilptb[B_SIZE];		/* *vilptbf[B_SIZE];	*/
	unsigned char	*poplnbuf[B_SIZE];
	unsigned char	f_name[50],tmp_str[20]; /* c_file[60],c_file0[60],ss[20]; */ /* file name and temoprary string*/

	unsigned short	i,j,k, offset, yr_tmp, cnum, lu_tmp, opt;
	short	kl,kr,ku,kd,ki,kj;  /* variables to set the "box" grids of buffer*/
	unsigned short	add_ir_grids /* irrigated rice*/ , add_sr_grids /* seasoned rice i.e rain-fed rice*/ , add_sc_grids /* shifting cultivation i guess*/;
	unsigned short	new_grids, newprice[CROP+1]; /* i suppose these are the new LU estimation grids and newprice of crops due to migration caused by food shortage and income fall*/



	unsigned int	vskip[V_MAX], vdemc,vdemr; /*I guess vskip is for ranking villages, vdemr is village demand of rice and vdemc is village demand of crops(maize)*/
	unsigned int	vsupr,vsupc; /*vsupr is village supply of rice and vsupc is village supply of crops(maize)*/


	float	area, rice_need, rice_need_area, new_supply, alpha, ur_need /*upland rice */, cr_need /*cultivatted rice*/ ;
	float	domestic_dem, rice_supply, inf;

	unsigned long dem, sup;

	
	
	yr=START_YR;
	/* READ ALL Data */

	/* PERMANENT DATA - themes don't change */
    if((fppopln=fopen("../../data/500m/img/bil/grid_pop_500_90.bil","rb"))==NULL){ /*rb = read binary file*/ /* population of start year i.e 1990*/
		printf("Can't open file grid_pop_500_90.bil\n"); exit(0);}

	if((fpelev=fopen("../../data/500m/img/bil/lp_dem_500.bil","rb"))==NULL){             /*elevation*/     
		printf("Can't open file lp_dem_50.bil\n"); exit(0);}

	if((fpslope=fopen("../../data/500m/img/bil/lp_sl_500.bil","rb"))==NULL){	/* slope in  % */ /* why slope is kept in percentage ? */
		printf("Can't open file lp_sl_500.bil\n"); exit(0);}

/*	if((fplu=fopen("../../data/500m/img/bil/rs_lu_500.bil","rb"))==NULL){		- Nagasawa RS area LU data */

	if((fplu=fopen("../../data/500m/img/bil/lp_lu_500.bil","rb"))==NULL){		/* Full LuangPrabang LU data */
		printf("Can't open file lp_lu_500.bil\n"); exit(0);}

	if((fprivbuf=fopen("../../data/500m/img/bil/lp_rivbuf_500.bil","rb"))==NULL){     /* river buffer */ /*where is it used ???*/
		printf("Can't open file lp_rivbuf_500.bil.bil\n"); exit(0);}

/*	if((fpvil=fopen("../../data/500m/img/bil/vil_bnd_500fl.bil","rb"))==NULL){
		printf("Can't open file vil_bnd_500fl.bil\n"); exit(0);}	*/

	if((fpvil=fopen("../../data/500m/img/bil/vil_cluster.bil","rb"))==NULL){        /*village  cluster = agent*/
		printf("Can't open file vil_cluster.bil\n"); exit(0);}


	if((fpvilpt=fopen("../../data/500m/img/bil/lp_vil_500.bil","rb"))==NULL){   /* village as points. village distribution data*/   /*where is it used ?? */
		printf("Can't open file lp_vil_500.bil\n"); exit(0);}

/*	if((fpvilptbuf=fopen("../../data/500m/img/bil/lp_vilbuf_500.bil","rb"))==NULL){
		printf("Can't open file lp_vilbuf_500.bil\n"); exit(0);}	*/

	if((fproad=fopen("../../data/500m/img/bil/lp_road_500.bil","rb")) ==NULL){      /*roads */
		printf("Can't open lp_rd_500.bil\n"); exit(0);}

	printf("Finished Opening the files\n");


	/* now we will set file pointer to start of each spatial file. then we will set pointer to each row via for loop of all such files. 
	And since we have pointer to each row of all such files, we will read the data columnwise via for loop from each low of each file */

	offset=(LIN_START*PIX_TOT);   /*LIN start refer to rows*/ /*this  is 0 as LIN_START = 0 . offset is 0 means start from beginning*/

	fseek(fppopln,2*offset,SEEK_SET);   /*BUT OFFSET IS 0 ?????????????*/     /* 2 byte= unsigned short int ... 0 to 65,535 */
	fseek(fpelev,2*offset,SEEK_SET);      /*BUT OFFSET IS 0 ?????????????*/      /* 2 byte= unsigned short int ... 0 to 65,535 */
	fseek(fpslope,1*offset,SEEK_SET);                                           /* 1 byte = unsigned char ..... 0 to 255  */
	fseek(fplu,1*offset,SEEK_SET);                                                 /* 1 byte = unsigned char ..... 0 to 255  */ 
	fseek(fprivbuf,2*offset,SEEK_SET);                                          /* 2 byte= unsigned short int ... 0 to 65,535 */
	fseek(fpvil,4*offset,SEEK_SET);                                              /* 4 byte= unsigned t int ... 0 to 4,294,967,295 */
	fseek(fpvilpt,4*offset,SEEK_SET);                                           /* 4 byte= unsigned t int ... 0 to 4,294,967,295 */
/*	fseek(fpvilptbuf,2*offset,SEEK_SET);	*/
	fseek(fproad,1*offset,SEEK_SET);                                               /* 1 byte = unsigned char ..... 0 to 255  */

	offset=PIX_START;     /*PIX_START=0*/	/*Pix start refer to columns*/

	printf(" Now Started Reading the Spatial Data Files\n");
	printf("........0%%");
	yr_tmp=yr-START_YR;

	/*setting pointer to each row via for loop for each spatial file*/
	for(row=0;row<LINE;row++){
		fseek(fppopln,2*offset,SEEK_CUR);    /*BUT OFFSET IS 0 ?????????????*/
		fseek(fpelev,2*offset,SEEK_CUR);      /*BUT OFFSET IS 0 ?????????????*/ 
		fseek(fpslope,1*offset,SEEK_CUR);
		fseek(fplu,1*offset,SEEK_CUR);
		fseek(fprivbuf,2*offset,SEEK_CUR);
		fseek(fpvil,4*offset,SEEK_CUR);
		fseek(fpvilpt,4*offset,SEEK_CUR);
/*		fseek(fpvilptbuf,2*offset,SEEK_CUR);	*/
		fseek(fproad,1*offset,SEEK_CUR);

		/*And since we have pointer to each row of all such files, we will aread the data columnwise via for loop from each row of each file and STORE that data in the array */
		

		for(col=0;col<PIXEL;col++){
			fread(lubuf,sizeof(char),1,fplu);
				land[yr_tmp][row][col]=(unsigned int)(xatoi(lubuf,0,1)); /*3D array*/   /* why x is used with atoi ???????????????????????????*/

			fread(poplnbuf,sizeof(char),2,fppopln);
				popln[yr_tmp][row][col]=(unsigned int)(xatoi(poplnbuf,0,2)); /*3D array*/ 

			fread(elevbuf,sizeof(char),2,fpelev);                        /* this is for the existence of elevation at row,col*/        
				elev[row][col]=(unsigned int)(xatoi(elevbuf,0,2));

			fread(slopebuf,sizeof(char),1,fpslope);                    /* this is for the existence of slope at row,col*/
				slope[row][col]=(unsigned int)(xatoi(slopebuf,0,1)); 

			fread(rivbuffer,sizeof(char),2,fprivbuf);                   /* this is for the existence of river at row,col*/ 
				rivbuf[row][col]=(unsigned int)(xatoi(rivbuffer,0,2));

			fread(vilbuf,sizeof(char),4,fpvil);                       /* this is for the existence of villge at row,col*/   
				vil[row][col]=(unsigned int)(xatoi(vilbuf,0,4));

			fread(vilptb,sizeof(char),4,fpvilpt);                    /* this is for village point boundary i guess at row,col*/
				vilpt[row][col]=(unsigned int)(xatoi(vilptb,0,4));

/*			fread(vilptbf,sizeof(char),2,fpvilptbuf);
				vilptbuf[row][col]=(unsigned int)(xatoi(vilptbf,0,2)); */

			fread(roadbuf,sizeof(char),1,fproad);                       /* this is for the existence of road at row,col*/
				road[row][col]=(unsigned int)(xatoi(roadbuf,0,1));
			
			if(land[yr_tmp][row][col]<=0){		/* Added, so that model will run for the RS image area only */ /*here RS is remote sensing only as base map is RS in the beginning ?*/
					popln[yr_tmp][row][col]=0;
					elev[row][col]=0;
					slope[row][col]=0;
					rivbuf[row][col]=0;
					vil[row][col]=0;
					vilpt[row][col]=0;
	/*				vilptbuf[row][col]=0; */
					road[row][col]=0;
			}


		}
		offset=PIX_TOT-PIXEL;     /*again offset is zero*//* what is it and why ?????????????????????*/
		/* show_progress3(row);	*/
	}
	fclose(fppopln);	fclose(fpelev);		fclose(fpslope);	fclose(fplu);
	fclose(fprivbuf);	fclose(fpvil);		fclose(fpvilpt);	fclose(fproad);
/*	fclose(fpvilptbuf);	*/

		
	printf("Finished reading the Permanent data - spatial ones\n");

	/* CROP DATA -moved, to read as needed*/
		
	/*  TABLE DATA - from text files */

	/*  1 read average crop yields for the different years*/
	/* UNITS: ton/Ha 1990-2000, both inclusive */

		fptmp=fopen("../../data/500m/text/crop_avg_yld.txt","r");

		if(fptmp==NULL)
		{printf("Can't open crop_avg_yld.txt \n");	exit(0);}

		printf("Opened  crop_avg_yld.txt file\n");
		printf("Printing the values as read :\nYear\tCrop\tAvg_Yield\n");

		for(i=0;i<NO_YR;i++)
		{ /*NO_YR = 11*/
			fscanf(fptmp,"%d",&dum_i);     /* dum_i means what... does this menas year taken from file ? ... Is it dummy variable for i everywhere in code */
			yr_tmp=dum_i;      /* everytime we have to enter the year inform of dum_i . enter the value of dum_i for each crop*/
			printf("%d",yr_tmp);
			for(k=1;k<=CROP;k++){  /*CROP = 4*/
				fscanf(fptmp,"\t%f",&crop_avg_yld[i][k]);
			/* 	vskip[k]=NAAM(k,tmp_str);	just to get crop name  vkip[] is used here as dummy
				printf("\t%d\t%s\t%f",k,tmp_str,crop_avg_yld[i][k]);	*/
				printf("\t%d\t%f",k,crop_avg_yld[i][k]);
			}
			printf("\n"); /*this takes the prt to the next line i.e next yeaR*/
		}
		fclose(fptmp);
		printf("Closed  Crop_Avg_Yld.txt file\n");
		
	/*  2 read farmgate price for the crops, of each year - DATA needs to be corrected */
	/* UNITS: Kip/kg 1990-2000, both inclusive */

		fptmp=fopen("../../data/500m/text/price.txt","r");
		if(fptmp==NULL){printf("Can't open price.txt \n");	exit(0);}

		printf("Opened  price.txt file\n");
		printf("Printing the values as read :\nYear\tCrop\tfgprice\n");

		for(i=0;i<NO_YR;i++){ /*NO_YR = 11*/
			fscanf(fptmp,"%d",&dum_i);
			yr_tmp=dum_i;
			for(k=1;k<=CROP;k++){   /*CROP = 4*/
				fscanf(fptmp,"\t%d",&fgprice[i][k]);
				printf("%d\t%d\t%d\n",yr_tmp,k,fgprice[i][k]);   /*this \n takes the prt to the next line i.e next yeaR*/
			}
		}
		fclose(fptmp);
		printf("Closed  Price.txt file\n");

	/* 3	READ Population growth rate data */
		fptmp=fopen("../../data/500m/text/pop_rate_new.txt","r");
		if(fptmp==NULL){printf("Can't open pop_rate_new.txt \n");	exit(0);}
		printf("Opened  pop_rate_new.txt file\n");
		printf("Printing the values as read :\nYear\tgr_rate\n");
		for(i=0;i<NO_YR;i++){
			fscanf(fptmp,"%d\t%f",&dum_i,&gr_rate[i]);
			printf("%d\t%f\n",dum_i,gr_rate[i]);
		}
		fclose(fptmp);
		printf("Closed  pop_rate_new.txt file\n");

		
		
	/* 4	READ Inflation data */
		fptmp=fopen("../../data/500m/text/inflation_rate.txt","r");
		if(fptmp==NULL){printf("Can't open inflation_rate.txt \n");	exit(0);}

		printf("Opened  inflation_rate.txt file\n");
		printf("Printing the values as read :\nYear\tinf_rate\n");
		for(i=0;i<NO_YR;i++){
			fscanf(fptmp,"%d\t%f",&dum_i,&inf_rate[i]);
			printf("%d\t%f\n",dum_i,inf_rate[i]);
		}
		fclose(fptmp);
		printf("Closed  inflation_rate.txt file\n");
		
	
	
	
	/* ******************* CALCULATION STARTS ********************* */
	
	/* DATA that need only one run */
	/*  1-1	Calculating j_max  */
	j_max=LINE;		
	if(j_max<PIXEL)j_max=PIXEL;  /*why and where is it used ?? */       /*j_max is the no of grids to be taken in the inner for loop i guess*/
	
	/*  1-2 Buffering Road Distances and use it throughout the model(no change in roads is assumed) */
	
	for(row=0;row<LINE;row++){	/* road pass-row starts */
		for(col=0;col<PIXEL;col++){	/* road pass-col start */
			d_road[row][col]=-99;	/* out side the area of interest */
			if(vil[row][col]==0)goto NEXT5; /*i.e village does not exist at row,col*/	/* out of area based on village*/
			
		   /* CHECK the road values, to choose only major roads for buffereing, not all 
		   - the original data has only one value as 100 */
			if(road[row][col]!=0) /*i.e road exist at row,col i.e "road in the village as a land use"  */
				d_road[row][col]=-1;	/* the road itself */
			else {	/* 6a-s */   /*?????????????????????????*/
				d_road[row][col]=0; /*initialisation or simple logical value assignment ????*/ /*i guess it represents the immediate neighbour to road, hence we can go from  in to out of the square box creating buffers*/
				j=1;   /*initialisation*/

				while(d_road[row][col]==0 && j<20){	/* while2-s */ /* what is 2-s ?????? */
					/* d_road[row][col] is the dist to the Nearest Road */
					/*changed to limit to 10 instead of j_max, to limit searches for road to upto 10km from the grid i.e 500m x 20 = 10 km*/
					/*we are basically trying to create a box of 4 sides*/

					
					kl=col-j; kr=col+j; ku=row-j; kd=row+j;   /*current value of rows and columsn */
					if(kl<0)kl=0; if(kr>PIXEL-1)kr=PIXEL-1;   /*max and min value of column*/
					if(ku<0)ku=0; if(kd>LINE-1)kd=LINE-1;     /*max and min value of row*/

					ki=kl; /* 0 i.e initialisation */
					for(kj=ku;kj<=kd;kj++){	/* 4a-s LEFT col as ki is 0 */  /* what is 4a-s ????? does s= start and e == end */
						if(road[kj][ki]!=0){	/* 4b-s */ /*kj is for row and ki is for column*/
							d_road[row][col]=j;
							goto JUMP4;
						}	/* 4b-e */
					}	/* 4a-e */
					ki=kr;
					for(kj=ku;kj<=kd;kj++){	/* 4c-s right col */
						if(road[kj][ki]!=0){	/* 4d-s */
							d_road[row][col]=j;
							goto JUMP4;
						}	/* 4d-e */
					}	/* 4c-e */         
					kj=ku;
					for(ki=(kl+1);ki<=(kr-1);ki++){	/* 4e-s top row */
						if(road[kj][ki]!=0){	/* 4f-s */
							d_road[row][col]=j; 
							goto JUMP4;
						}	/* 4f-e */
					}	/* 4e-e */
					kj=kd;
					for(ki=(kl+1);ki<=(kr-1);ki++){	/* 4g-s bottom row */
						if(road[kj][ki]!=0){	/* 4h-s */
							d_road[row][col]=j;
							goto JUMP4;
						}	/* 4h-e */
					}	/* 4g-e */
					j++;
				}	/* while2-e */
			JUMP4:
			dum=0;
			}	/* 6a-e */
			NEXT5:
			dum=0;
		}	/* road pass-col ends */
	}	/* road pass-row ends */

	/* CREATE the output folder 
	if(system("dir op1") ) system("mkdir op1");	*/

	strcpy(f_name, "op1/road_buf.bil"); 
	if((fproad=fopen(f_name,"w")) ==NULL){
		system("mkdir  op1"); /*what is system ???*/
	}
	if((fproad=fopen(f_name,"w")) ==NULL){
		printf("Can't open %s\n",f_name); exit(0);
	}


	/*	strcpy(f_name, DIR_STR); strcat(f_name,"/road.buf.bil");
		if((fproad=fopen(f_name,"w")) ==NULL){
			printf("Please Give the Folder name for Outputs\n");
			system(" set /P VAR= ");
			printf("Please reconfirm again\t:"); scanf("%s",&tmp_str);
			system("mkdir  %VAR%"); 
		}
		strcpy(DIR_STR, tmp_str);
		strcpy(f_name, DIR_STR); strcat(f_name,"/road.buf.bil");
		if((fproad=fopen(f_name,"w")) ==NULL){
			printf("Can't open %s\n",f_name); exit(0);
		}
	*/

	printf("Started writing Output of Road Buffer to: %s\n",f_name);
		for(row=0; row<LINE; row++)	{
			for(col=0;col<PIXEL;col++)	{
				print_buf[col]=d_road[row][col];  /*this is done to create output in BIL band interweaved by line format*/
			}
			fwrite(print_buf,sizeof(char),PIXEL,fproad);
		}
	fclose(fproad);

	strcpy(f_name,"op1/road_buf");strcat(f_name,".hdr");
	hdr_prn(1, 1, f_name);	/* 	dat=1; - sizeof(short); n_bands=1;*/ /* ................................ FUNCTION CALL @ p2............................... */

	printf("Finished Calculating Road Buffers\n");
	
	
/*  1-3  ********************************************************** */
/* ReNUmber the LU Map 	- use the current LU, with elev and slope*/ /*this part of hard coding is required as we have the knowledge of LU lcasses already from an external source like RS image or field study.*/ /*for our case we can gather such details from other's research papers and earthgeo sites*/
	for(row=0;row<LINE;row++){                   /*we use elev and slope as shifting cult requires a certain elev and slope as it usually occurs in mountain regions*/
		for(col=0;col<PIXEL;col++){
			lu_tmp =land[0][row][col];     /*i guess 0 here represents the start year. i suupose base year is 1989 and start year is 1990 ???????????*/
			land[0][row][col]=new_LU(lu_tmp); /* ................................ FUNCTION CALL @ p1 with each value of row,col............................... */
		}
	}
	
	yr=START_YR; /*1990*/	i_to_a(yr,tmp_str); /*to write the year in file name*/ /* ................................ FUNCTION CALL @ p1 ....... */
	strcpy(f_name,"op1/final_lu_"); strcat(f_name,tmp_str); strcat(f_name,".bil");
	if((fplu=fopen(f_name,"wb"))==NULL){printf("Can't open %s\n",f_name); exit(0);}    /*wb means write in binary file*/

	for(row=0;row<LINE;row++){
		for(col=0;col<PIXEL;col++){		/* band 1 - New LU values */
		/* 	print_buf[col]=land[yr-START_YR][row][col];	*/
		print_buf[0]=land[yr-START_YR][row][col]; /*yr-START_YR = 0 */
		fwrite(print_buf,sizeof(char),1,fplu); /* size_t fwrite ( const void * ptr, size_t size, size_t count, FILE * stream );Writes an array of count elements, each one with a size of size bytes, from the block of memory pointed by ptr to the current position in the stream.*/
		}
	}
	fclose(fplu);
	
	strcpy(f_name,"op1/final_lu_"); strcat(f_name,tmp_str); strcat(f_name,".hdr");
	hdr_prn(1, 1, f_name);	/* 	dat=1; - sizeof(char); n_bands=1;*/

/*  1-4  ********************************************************** */
	/* Calculate the Base Year Supply and Income Map */

	yr=START_YR; /*1990*/
	                     
	read_cropdata();             /* ................................ FUNCTION CALL @ p1............................... */      
			/* Reads all the Crop Data files and returns crop yield i.e cropyld[y][lun][row][col]	y=0,1,2,3; lun=0 to CROP;	*/
		
	vil_val();	/* returns values related to the Village */ /* ................................ FUNCTION CALL @ p2............................... */

	printf("Started Calculating & Writing Output of %d Income Values (in 1000 kips): %s\n",yr,f_name);
	printf("........0%%");

	i_to_a(yr,tmp_str);
	strcpy(f_name,"op1/inc_map_");    /*where do you use income  map in the model. how is it calculated ??*/
	strcat(f_name,tmp_str);
	strcat(f_name,".bil");
	if((fp_inc=fopen(f_name,"w")) ==NULL) 
		{printf("Can't open %s\n",f_name); exit(0);}

		/* Max_opt=0; */opt=0;	n_option=0;
	/*	dev_cost[0]=0; Areal_Change[0]=0;	onfarm_fac[opt]=offfarm_fac[opt]=1.0;	*/  /* what are these ??? */

	/*Initialize data */
	for(row=0;row<LINE;row++){
		for(col=0;col<PIXEL;col++){
			HH_Inc_grid[0][row][col]=HH_Inc_grid[1][row][col]=HH_Inc_grid[2][row][col]=0; /*initialisation *//*what is HH_Inc_grid, 0,1,2 ???? */ /*  NEED HH_Inc_grid[yr][row][col] --> yr=0,1,2 for as the Migration criteria */
		}
	}

	for(k=0;k<=vilmax;k++)
	{	
		for(j=0;j<=CROP;j++)    /*initialisation */
		{ vilsup[k][j]=0;}   /*vilsup[vil no][crop]*/
	}

	for(yr=START_YR;yr<END_YR;yr++){ 
		for(cnum=1;cnum<=CROP;cnum++){          /*  CROP= 4	:: Irrigated Rice, Season Rice, Upland Rice and Crop-Maize  */
			supply[yr-START_YR][cnum]=0; /*initialisation */
			prodn_area[yr-START_YR][cnum]=0; /*initialisation */
		}
	}
	for(row=0;row<LINE;row++){
		for(col=0;col<PIXEL;col++){
			
			/*	agri_f = 100;	on_f=0;	off_f=0; these are values in percentages */  /*onfarm_fac[opt]=offfarm_fac[opt]= 0;	*/ 
		
			gen_cost[opt]=0;  revenue[opt]=0;  /*opt menas option. there are 20 options for villages*/
			
			supply_inc(opt); /* ................................ FUNCTION CALL @ p1............................... */		
			
			/*this function is to calculate the Income for each Agri  LU option based on its specific crop-rotation cycles + Supply data*/
			
			/*	Assuming a family of 6 per HH */
			
			/*  NEED HH_Inc_grid[yr][row][col] --> yr=0,1,2 for as the Migration criteria */

			if(popln[yr-START_YR][row][col]<=0)
				HH_Inc_grid[0][row][col]=0;

			else HH_Inc_grid[0][row][col]=(int)(inc_yr[yr-START_YR][row][col]/(popln[yr-START_YR][row][col]/6.0));

			/* this is done to initialize the values for the first time frame of yr-1, and yr-2 years */

			HH_Inc_grid[2][row][col]=HH_Inc_grid[0][row][col];   /*???????????*/
			HH_Inc_grid[1][row][col]=HH_Inc_grid[0][row][col];   /*???????????*/
			
			print_buf[col]=inc_yr[yr-START_YR][row][col];      /*???????????*/

			/* printf("%d\t",inc_opt[0]); */
		}

	/*	show_progress3(row);
		 printf("\n"); */
		fwrite(print_buf,sizeof(short),PIXEL,fp_inc);
	}

	fclose(fp_inc);

	strcpy(f_name,"op1/inc_map_"); strcat(f_name,tmp_str); strcat(f_name,".hdr");
	hdr_prn(2, 1, f_name);	/* 	dat=2; - sizeof(short); n_bands=1;*/ /* ................................ FUNCTION CALL @ p2............................... */
	printf("Finished Calculating %d Income Map\n",yr);
	
	
	/* **********************************************
			sri sri
	********************************************** */
	/* from here - FOR ALL YEARS  */	

	for(row=0;row<LINE;row++){
		for(col=0;col<PIXEL;col++){
			In_Mig[row][col]=Out_Mig[row][col]=0; /*initialisation*/
		}
	}

	printf("///* *************** sri *********************** *///\n");
	printf("\tSTARTED CALCULATING ON A YEARLY BASIS \n");

	for(yr=START_YR+1;yr<END_YR;yr++){ /* START_YR+1 means 1991*/
		
		/* initialize the land data */
		for(row=0;row<LINE;row++){
			for(col=0;col<PIXEL;col++){
				land[yr-START_YR][row][col]=0; /*initialisation*/
			}
		}
		for(i=0;i<=CROP;i++){  /*CROP= 4*/
			supply[yr-START_YR][i]=0; /*initialisation*/
			if(yr==START_YR+1) /*i.e the the year after start year or 1990*/
				price[yr-START_YR-1][i]=fgprice[yr-START_YR-1][i]; /*from here the preliminary priting of output starts ...............*/
		}
		printf("///* ******************************************* *///\n");
		printf("\tStarted the CALCULATIONS for %d\n\tFinished Initializing land[%d][r][c]\n",yr,(yr-START_YR));

		/* ******************* SET ONE ************************** */

		printf("Started Rural income part - Set Two of yr %d\n",yr); /*what do you mean by set two here when the comment says we are in SET ONE ???*/

		/*	Crop Data to read */
		read_cropdata();  /*reads crop data from bile files of four kind of crops*//* ................................ FUNCTION CALL @ p1.............................. */
			/* Reads all the Crop Data files and returns 	cropyld[y][crn][row][col]	y=0,1,2,3; crn=1 to CROP;	*/
		
		popln_sum=0;
		vil_val();	 /* returns the total number of villges*//* returns values related to the Village */ /* ................................ FUNCTION CALL @ p2............................... */

		printf("Total Poulation at Start of Year %d is %d\n", yr, popln_sum);
		vdemr=0; popln_sum=0;    /* what is vdemr  ... is it village dem of rice ????*/ /* initialisation */
		for(i=1;i<=vilmax;i++){   /* vilmax  is The Total number of Villages */
			vdemr+=vilgrids[i];  /*does vdemr means sum of demand in working village grids, coz if so .... */
			popln_sum+=vilpop[i];
			printf("No. of Grids in respective LUs are\nVil\tLU[20]\tLU[21]\tLU[22]\tLU[23]\tLU[24]\n");
			printf("%d \t%d \t%d \t%d \t%d \t%d\n",vilsno[i],villu[i][0],villu[i][1],villu[i][2],villu[i][3],villu[i][4]); /*0 to 4 coz there 5 LUs of use*/
		}

		printf("Total Working grids are %d & Total Popln is %d \n", vdemr, popln_sum);
		
		/* Check the for Rice Supply-Demand */
		domestic_dem = popln_sum * 0.6*365.0 / 1000.0;	/* Consumption of 600gms/person/day; dem is in tons*/

		rice_supply = supply[yr-START_YR-1][1] + supply[yr-START_YR-1][2] + supply[yr-START_YR-1][3]; /*what is 1,2,3 here*/ /*i guess it is irrigated rice and seasoned rice and upland rice */
		/* assuming external demand = 0 */

		if(domestic_dem>(0.99*rice_supply)) {	/* 1% is part of loss */
			rice_need = domestic_dem-(0.99*rice_supply);
			printf("Domestic Dem: %ftons\t Rice_supply: %ftons\t Rice_Need: %f\n",domestic_dem,(0.9*rice_supply),rice_need); /*what is the diff btwn domestic demand and rice need ???*/
		}
		/* else { rice_need = 0.05*rice_supply;	 ** assumed that still there is a 5%inc in rice prodn. for a start **
		printf("Rice_need: %f tons \n",rice_need);
		}
		*/
		
		/* Assume Increase in Irrigated Area on a year on year basis at 15% */
		if(yr<=1995)alpha=0.10;	/*10% inc */
		else if(yr==1996)alpha=0.57;	/* 57% inc */
		else if(yr==1997)alpha=0;	/* -18 inc */
		else if(yr==1998)alpha=0.77;	/* 77% inc */
		else if(yr==1999)alpha=0.44;	/* 44% inc */
		else alpha=0;

		area= GFAC*(CELL/100)*(CELL/100);	/* units are Ha as CELL is in meters (UTM)*/

		new_grids = (int)((alpha * prodn_area[yr-START_YR-1][1] / area )+0.5) ;  /*why 0.5 ??? */

		/* if(new_grids>0) */ add_ir_grids=allot_ir(new_grids);		/** function to alot New IR grids **//* ................................ TWO FUNCTION CALL @ p2............................... */
		
		new_supply=add_ir_grids*area*crop_avg_yld[yr-START_YR-1][1]; /* 1 here i guess menas irrigated rice */
		printf("\t\tAfter Increasing IR area\nDomestic Dem: %ftons \t Rice_supply(t-1): %ftons New_IR_Supply: %f\n",domestic_dem,rice_supply,new_supply);

		if(domestic_dem>0.99*(rice_supply+new_supply)) {
			rice_supply+=new_supply;
			rice_need = domestic_dem-(0.99*rice_supply);
			printf("So, further Rice_need after IR addition is: %f tons \n",rice_need);
		}
		else goto SC_SIM;	/*if rice is satisfied, then go to Shifting Cultivation Simulation */
		
		rice_need_area = (rice_need / crop_avg_yld[yr-START_YR-1][2]); /* 2 i guess here means seasoned rice*/

			/* but limit it to not more than 10%rise in a year */

		if(rice_need_area>(0.1*prodn_area[yr-START_YR-1][2])){ /* 2 i guess here means seasoned rice*/ /* but limit it to not more than 10%rise in a year */
			printf("As rice_need_area (=%f) exceeded the permissible limit for SR area increase,\n it is changed to %f\n",rice_need_area,(0.1*prodn_area[yr-START_YR-1][2]));
			rice_need_area=(0.1*prodn_area[yr-START_YR-1][2]); /*why 0.1 ???*/ /* but limit it to not more than 10%rise in a year */
		}

		new_grids = (int)((rice_need_area/ area)+0.5); /*why 0.5. i guess htis is for utm to grid conversion ???*/

		if(new_grids>0) 
			
		add_sr_grids=allot_sr(new_grids);	/* ................................ FUNCTION CALL @ p2............/*	/** function to alot New SR grids **/
		
		new_supply=add_sr_grids*area*crop_avg_yld[yr-START_YR-1][2]; /* 2 i guess here means seasoned rice */

		printf("\t\tAfter Increasing SR area\nDomestic Dem: %ftons \t Rice_supply(wirh IR): %ftons New_SR_Supply: %f\n",domestic_dem,rice_supply,new_supply);

		if(domestic_dem>(rice_supply+new_supply)) 
		{
			rice_supply+=new_supply;
			rice_need = domestic_dem-rice_supply;
			printf("So, further Rice_need after SR addition is: %d tons \n",rice_need);
		}
		else goto SC_SIM;	/*LINE 593 *//*if rice is satisfied, then go to Shifting Cultivation Simulation */

		/*............................. SET ONE ENDS .....................................................................................................................*/
		
		/* ************************************************ SET - TWO ************************ ***************************************************************/
		

                SC_SIM:      /* this is shifting cultivation simulation i guess*/

		for(i=0;i<V_MAX;i++) /*V_MAX =2500 ; assumed a max of 2500 villages - as clusters is only 7 */

		{ 
			vskip[i]=0; 
		    cycle[i]=0;  /* shifting cultivation cycle ; initialisation*/
		} /* refer global from V_MAX */

		for(i=1;i<=vilmax;i++)  /* vilmax = 7 clusters or agnets*/
		
		{ 
			printf("Now working on the Village No.%d, vilsno[%d]:%d\n",i,i,vilsno[i]);
			dum2=i;
			
			/* Check the for Rice Supply-Demand */

			vdemr=vilpop[dum2]*0.6*365.0/1000.0; /*dum2=i*/	/* Consumption of 600gms/person/day rice; dem is in tons*/
			vdemc=vilpop[dum2]*0.05*365.0/1000.0; /*dum2=i*/	/* i guess Consumption of 50gms/person/day of other crop ; dem is in tons*/
				/* Consumption of 100gms/person/day - CROP; dem is in tons ** Changed to 50gm*/

			vsupr=0; /*Village Supply of Rice*/
			area= GFAC*(CELL/100)*(CELL/100);	/* units are Ha as CELL is in meters (UTM)*/

			yr_tmp=yr-START_YR-1; /* START_YR-1=1989*/

			/*dum2 = i as mentioned above*/
			/*villu(grids of a particular lu) x area x crop_avg_yld = supply */
			/*in below , why 1,1,2,3,4  AND 1,2,2,3,4*/

			vsupr+=(int)(villu[dum2][1]*area*crop_avg_yld[yr_tmp][1]); /*dum2=i*//* for IR */ 
			vsupr+=(int)(villu[dum2][1]*area*crop_avg_yld[yr_tmp][2]); /*dum2=i*//* for SR in IR areas */
			vsupr+=(int)(villu[dum2][2]*area*crop_avg_yld[yr_tmp][2]); /*dum2=i*//* for SR */
			vsupr+=(int)(villu[dum2][3]*area*crop_avg_yld[yr_tmp][3]); /*dum2=i*/ /* for UR */
			vsupc= (int)(villu[dum2][4]*area*crop_avg_yld[yr_tmp][4]); /*dum2=i*//* for Crop-Maize */  /*why is it not added everytime ??? like other 4 above ????????*/ 

/* condition1.*/if(vsupr>=vdemr && vsupc>=vdemc)   /* vsupr and vdemr are both for rice*/ /* vsupc and vdemc are both  for crop-maize*/
			{ 
				if ((villu[dum2][3]+villu[dum2][4])<=0)  /*ur +cfrop-maize*/ /*3 is ur and 4 is other crop ie. maize*/
					goto NV1;
				else 
				{
					ur_gr=villu[dum2][3]; /*dum2=i*//*upland rice grids*/
					cr_gr=villu[dum2][4]; /*dum2=i*//*crop-maize grids i suppose ???*/
					goto SC_C;	/* getting the spatial shift of the crop area */
				}
			}

/* condition2.*/else if(vsupr>=vdemr)
              {
				/*only crop in SC - spatial shift plus inc in crop area, if any; rice satisfied from IR&SR*/
				ur_gr=villu[dum2][3];/*dum2=i*/ /* upland_rice grids*/
				/*chk vsupc & vdemc - calc cr_area*/
				cr_need=vdemc-vsupc;	/*in tons*//*i guess this is for maize*/
				rice_need_area=cr_need / crop_avg_yld[yr_tmp][4];
				cr_gr = (int)( (rice_need_area/ area)+0.6); /*this the crop-maize grids i guess*/
				cr_gr+=villu[dum2][4]; /*dum2=i*//*final summation count of crop-maize grids i guess*/ /* cr_gr represents grids for need of crop-maize */
				if(cr_gr>(1.15*villu[dum2][4])){
				printf("As Crop_need_area (=%d) exceeded the permissible limit for CR area increase in Vil[%d] %d,\n it is changed to %d\n",cr_gr,dum2,vilsno[dum2],((int)(1.15*villu[dum2][4])) );
				cr_gr=(1.15*villu[dum2][4]);/*dum2=i*/ /*crop-maize*/
				}
				goto SC_C;
			}
/* condition3.*/else {	
				/* here considering total area for SC as a combi of SC_UR and SC_CR,
				so will chk UR S-D and crop S-D while allocating new grids only */
				ur_need=vdemr-vsupr;	/*in tons*/
				rice_need_area=ur_need / crop_avg_yld[yr_tmp][3];
				ur_gr = (int)( (rice_need_area/ area)+0.6);
				ur_gr+=villu[dum2][3];/*dum2=i*/

				cr_need=vdemc-vsupc;	/*in tons*/
				rice_need_area=cr_need / crop_avg_yld[yr_tmp][4];
				cr_gr = (int)( (rice_need_area/ area)+0.6);
				cr_gr+=villu[dum2][4];
				if(cr_gr>(1.15*villu[dum2][4]))/*dum2=i*/
				{
				printf("As Crop_need_area (=%d) exceeded the permissible limit for CR area increase in Vil[%d] %d,\n it is changed to %d\n",cr_gr,dum2,vilsno[dum2],((int)(1.15*villu[dum2][4])) ); /*why int is used here ????*/
				cr_gr=(1.15*villu[dum2][4]); /*dum2=i*/
				}
			}

			SC_C:
			/* get total area to spatially assign, calc areas not to touch in the vil, 	calc cycle and allot randomly in clusters of 4min grids 
			-- removed clustering part, so each grid is randon now, so using "allot_sc2" instead of allot_sc */

			new_grids=ur_gr+cr_gr; /*upland rice + crop_maize grids*/
			if(new_grids>vilgrids[dum2])/*dum2=i*/
				new_grids=vilgrids[dum2]; /*dum2=i*/
			if(new_grids<=0)
				cycle[dum2]=0; /*dum2=i*/

			else cycle[dum2]=(int)((villu[dum2][0]+villu[dum2][3]+villu[dum2][4])/new_grids); /* dum2=i */ /*what is villu[dum2][0]??????? i guess its the available area that can be used for as land use*/ /*villu[dum2][3]= ur + villu[dum2][4] = crop maize */

			if(cycle[dum2]<3 && cycle[dum2]>0){ /*dum2=i*/
				cycle[dum2]=3;	/*min 3yrs */ /*dum2=i*/
				new_grids = (int)((villu[dum2][0]+villu[dum2][3]+villu[dum2][4])/cycle[dum2]); /*dum2=i*/   /*what is villu[dum2][0]??????? i guess its the available area that can be used for as land use*/ /*villu[dum2][3]= ur + villu[dum2][4] = crop maize */
			}
			
			/** spatial allocation 
			if(new_grids>0) add_sc_grids=allot_sc(new_grids,dum2);		function to alot New SC grids **/

			if(new_grids>0) 
				add_sc_grids=allot_sc2(new_grids,dum2); /*dum2=i*//* ................................FUNCTION CALL @ p2............................... */

			NV1:
			dum2=0;
		}
					/* WRITING LU MAP */
			/* *** Format of LU as is used in the program i.e. MMmmP where MM<mm *** */
		i_to_a(yr,tmp_str); /* ................................FUNCTION CALL @ p1............................... */
		strcpy(f_name,"op1/final_lu_"); strcat(f_name,tmp_str); strcat(f_name,".bil");
		if((fplu=fopen(f_name,"wb"))==NULL){printf("Can't open %s\n",f_name); exit(0);}

		for(row=0;row<LINE;row++){
			for(col=0;col<PIXEL;col++){		/* band 1 - New LU values */
				if (land[yr-START_YR][row][col]==0 && land[yr-START_YR-1][row][col]==20){ /*0 here means have been initialzed already , 20 means available*//*LU numbers are 20-avail SC Area; 21-SR+IR; 22-SR; 23-UR; 24-Crop=Maize*/
					land[yr-START_YR][row][col]=20;	/* areas of avail SC area that is not used this year */ /*only 20 (avail) can be used to add grids to sc*/
				}
				else if(land[yr-START_YR][row][col]<20){
						/* || land[yr-START_YR][row][col]>24) -- this is not needed */
					land[yr-START_YR][row][col]=land[yr-START_YR-1][row][col];
				}
			/*	print_buf[col]=land[yr-START_YR][row][col];	*/
				print_buf[0]=land[yr-START_YR][row][col]; /*what is the use of print_buf*/
				fwrite(print_buf,sizeof(char),1,fplu);
			}
			/*fwrite(print_buf,sizeof(char),PIXEL,fplu);*/
		}
		fclose(fplu);
		
		strcpy(f_name,"op1/final_lu_"); 
		strcat(f_name,tmp_str); 
		strcat(f_name,".hdr");
		hdr_prn(1, 1, f_name);	/* 	dat=1; - sizeof(char); n_bands=1;*/ /* ................................ FUNCTION CALL @ p2............................... */

				/*  INCOME CALCULATION */
		printf("Started Calculating & Writing Output of %d Income Values (in 1000 kips): %s\n",yr,f_name);
		printf("........0%%");

		i_to_a(yr,tmp_str);
		strcpy(f_name,"op1/inc_map_");
		strcat(f_name,tmp_str); 
		strcat(f_name,".bil");

		if((fp_inc=fopen(f_name,"w")) ==NULL) 
			{printf("Can't open %s\n",f_name); exit(0);}

		/* Max_opt=0; */opt=0;	n_option=0;
		/*	dev_cost[0]=0; Areal_Change[0]=0;	onfarm_fac[opt]=offfarm_fac[opt]=1.0;	*/

		/*Initialize data */
		for(k=0;k<=vilmax;k++)/*vilmax=7 i guess*/
		   {	
			for(j=0;j<=CROP;j++) /*CROP =4	Irrigated Rice, Season Rice, Upland Rice and Crop-Maize */
			{ 
				vilsup[k][j]=0; /* vilsup[][] has vilalge serial no and crop number as array indices */
			}
		}
		for(k=START_YR;k<END_YR;k++) /*START_YR = 1990;END_YR=2000*/
		{
			for(cnum=1;cnum<=CROP;cnum++)
			{
				supply[k-START_YR][cnum]=0; 
				prodn_area[k-START_YR][cnum]=0;
			}
		}

		for(row=0;row<LINE;row++){
			for(col=0;col<PIXEL;col++){
				
				/*	agri_f = 100;	on_f=0;	off_f=0;	these are values in percentages */
			
				gen_cost[opt]=0;
				
				revenue[opt]=0;
				
				supply_inc(opt);	/* ................................FUNCTION CALL @ p1............................... */	
				
				/*this function,supply_inc(opt), is to calculate the Income for each Agri LU option based on its specific crop-rotation cycles + Supply data*/ 
				
				/*	Assuming a family of 6 per HH 	 */			

				if(popln[yr-START_YR-1][row][col]<=0)
					HH_Inc_grid[0][row][col]=0;
				else HH_Inc_grid[0][row][col]=inc_yr[yr-START_YR][row][col]/(popln[yr-START_YR-1][row][col]/6.0); /*wha t is 0 in HH_Inc_grid here ??*/
				
				print_buf[col]=inc_yr[yr-START_YR][row][col];
			}
		/* 	show_progress3(row);
			printf("\n"); */
			fwrite(print_buf,sizeof(short),PIXEL,fp_inc);
		}

		fclose(fp_inc);
		strcpy(f_name,"op1/inc_map_"); 
		strcat(f_name,tmp_str); 
		strcat(f_name,".hdr");
		hdr_prn(2, 1, f_name);	/* 	dat=2; - sizeof(short); n_bands=1;*/
		printf("Finished Calculating %d Income Map\n",yr);


		/*............................. SET TWO ENDS .....................................................................................................................*/


		/* *************** SET 3 PRICE CALC .... Write output as year file - contents are Crop, Demand, Supply, Price-old, Price New ***** */


		strcpy(f_name,"op1/dem-sup_");

		i_to_a(yr,tmp_str);
		strcat(f_name,tmp_str); 
		strcat(f_name,".txt");
		if((fptmp=fopen(f_name,"w")) ==NULL) {printf("Can't open %s\n",f_name); exit(0);}

		printf("Started writing Demand-Supply-Price Output to %s\n",f_name);

		/*now printing in main output file/terminal where all the output is listed*/
			printf("UNITS: in 1000 Metric ton & in Bahts\n");
			printf("Crop\tDomestic_Demand\tProduction\t");
			printf("fgPrice[yr]\tLast Yr Price\tCalc Price[yr]\n");

       /*now printing in sub output file in op1 folder where all the sub outputs are listed */
		    fprintf(fptmp,"UNITS: in 1000 Metric ton & in Bahts\n");
		    fprintf(fptmp,"Crop\tDomestic_Demand\tProduction\t");
		    fprintf(fptmp,"fgPrice[yr]\tLast Yr Price\tCalc Price[yr]\n");

			inf=inf_rate[yr-START_YR];	/*inflation rate*/ /*float */
			
		/* for rice */
			dem = popln_sum * 0.6*365.0 / 1000.0;	/* Consumption of 600gms/person/day; dem is in tons*/ /* crop 1 = cnum=1= i_rice;crop 2 = cnum=2= s_rice;crop 3 = cnum=3= u_rice */
			sup = supply[yr-START_YR-1][1] + supply[yr-START_YR-1][2] + supply[yr-START_YR-1][3]; /* supply */ /* START_YR-1=1989*/ /*supply[yr-START_YR][cnum]*//* crop 1 = cnum=1= i_rice;crop 2 = cnum=2= s_rice;crop 3 = cnum=3= u_rice */
			cnum=1; /*i_rice*/
			/* graded increase in prices considering the inflation factoring  */
			if(sup<=0){	/* dummy price levels based on inf. */
				newprice[cnum]=price[yr-START_YR-1][cnum]*(1+(inf/100.0));
			}

			else if(sup<=0.7*dem){	/* dem-sup ratio */ /*how demand supply ratio i.e 0.7 is determined ????*/
				newprice[cnum]=price[yr-START_YR-1][cnum]*(dem/sup);
				if(inf>10.0) /* why 10 ??*/
					newprice[cnum]*=(1+(inf/100.0)); /*why 1+(inf/100.0)?? */
			}
			else if(sup<=0.9*dem){	/* 1.5*inf */  /*how demand supply ratio i.e 0.9 is determined ????*/
				newprice[cnum]=price[yr-START_YR-1][cnum]*(1+(inf*1.5/100.0)); /*why 1+(inf*1.5/100.0)?? */
			}
			else if(sup<=1.1*dem){	/* only inf. */ /*how demand supply ratio i.e 1.1 is determined ????*/
				newprice[cnum]=price[yr-START_YR-1][cnum]*(1+(inf/100.0)); /*why 1+(inf/100.0)?? */
			}
			else {	/* supply is more than 10% of the demand, then dem-sup ratio */
				newprice[cnum]=price[yr-START_YR-1][cnum]*(dem/sup);
				if(inf>10.0)newprice[cnum]*=(1+(inf/100.0));
			}

			if(newprice[cnum]>(1.2*price[yr-START_YR-1][cnum])) /*1.2 ????? */

			{	newprice[cnum]=1.2*price[yr-START_YR-1][cnum];
				if(inf>10.0)
					newprice[cnum]*=(1+(inf/100.0));
			}
			else if(newprice[cnum]<0.8*price[yr-START_YR-1][cnum]) /*0.8 ?*/
			{
				newprice[cnum]=0.8*price[yr-START_YR-1][cnum];
				if(inf>10.0)
					newprice[cnum]*=(1+(inf/100.0));	
			}
			/* price fluctuation limited to a 20% band */
			newprice[2]=newprice[3]=newprice[1]; /*after if and else if ..i  guess this is the condition due to less data available*/

			for(cnum=1;cnum<=3;cnum++) /*for all types of rice i.e crop 1 = cnum=1= i_rice;crop 2 = cnum=2= s_rice;crop 3 = cnum=3= u_rice */
			{		
			     price[yr-START_YR][cnum]=newprice[cnum];
				 printf("%d\t%d\t%d\t",cnum,dem,supply[yr-START_YR][cnum]);
				 printf("%d\t%d\t%d\n",fgprice[yr-START_YR][cnum],price[yr-START_YR-1][cnum],price[yr-START_YR][cnum]);

			fprintf(fptmp,"%d\t%d\t%d\t",cnum,dem,supply[yr-START_YR][cnum]);
			fprintf(fptmp,"%d\t%d\t%d\n",fgprice[yr-START_YR][cnum],price[yr-START_YR-1][cnum],price[yr-START_YR][cnum]);

			}

			/*for Crop */ /*i guess comment on the left means maize-crop as cnum=4; */
			dem = popln_sum * 0.05*365.0 / 1000.0;	/* Consumption of 100gms/person/day - reduced to 50gms; dem is in tons*/
			sup = supply[yr-START_YR-1][4]; /* crop 1 = cnum=1= i_rice;crop 2 = cnum=2= s_rice; crop 4 = cnum=4= maize */
						cnum=4; /*maize*/
			/* graded increase in prices considering the inflation factoring  */
			if(sup<=0){	/* dummy price levels based on inf. */
				newprice[cnum]=price[yr-START_YR-1][cnum]*(1+(inf/100.0));
			}
			else if(sup<=0.7*dem){	/* dem-sup ratio */
				newprice[cnum]=price[yr-START_YR-1][cnum]*(dem/sup);
				if(inf>10.0)newprice[cnum]*=(1+(inf/100.0));
			}
			else if(sup<=0.9*dem){	/* 1.5*inf */
				newprice[cnum]=price[yr-START_YR-1][cnum]*(1+(inf*1.5/100.0));
			}
			else if(sup<=1.1*dem){	/* only inf. */
				newprice[cnum]=price[yr-START_YR-1][cnum]*(1+(inf/100.0));
			}
			else {	/* supply is more than 10% of the demand, then dem-sup ratio */
				newprice[cnum]=price[yr-START_YR-1][cnum]*dem/sup;
				if(inf>10.0)newprice[cnum]*=(1+(inf/100.0));
			}

			if(newprice[cnum]>(1.2*price[yr-START_YR-1][cnum]))
			{  	newprice[cnum]=1.2*price[yr-START_YR-1][cnum];
				if(inf>10.0)newprice[cnum]*=(1+(inf/100.0));
			}
			else if(newprice[cnum]<0.8*price[yr-START_YR-1][cnum])
			{	newprice[cnum]=0.8*price[yr-START_YR-1][cnum];
				if(inf>10.0)newprice[cnum]*=(1+(inf/100.0));
			}
			/* price fluctuation limited to a 20% band */			
			cnum=4; /*maize*/

			    price[yr-START_YR][cnum]=newprice[cnum];
				printf("%d\t%d\t%d\t",cnum,dem,supply[yr-START_YR][cnum]);
				printf("%d\t%d\t%d\n",fgprice[yr-START_YR][cnum],price[yr-START_YR-1][cnum],price[yr-START_YR][cnum]);

			fprintf(fptmp,"%d\t%d\t%d\t",cnum,dem,supply[yr-START_YR][cnum]);
			fprintf(fptmp,"%d\t%d\t%d\n",fgprice[yr-START_YR][cnum],price[yr-START_YR-1][cnum],price[yr-START_YR][cnum]);
			
			fclose(fptmp);

			/*............................. SET THREE ENDS .....................................................................................................................*/



		/* ***************************SET 4 *************************** */





		/* SET 4 - MIGRATION PART 

		migration();  */
		

		 /* UPDATING HH_Inc_grid[y][row][col] --> y=0,1,2 (yr-1, yr-2 and yr-3) for as the Migration criteria */ /*what does this comment means ??*/
		for(row=0; row<LINE; row++){
			for(col=0; col<PIXEL; col++){
				HH_Inc_grid[2][row][col]=HH_Inc_grid[1][row][col]; /*whar is 0,1,2 here and why the value swap ??*/ /*i gues 0,1,2 is for migration years*/
				HH_Inc_grid[1][row][col]=HH_Inc_grid[0][row][col];/*whar is 0,1,2 here and why the value swap ??*/
				
				if(popln[yr-START_YR][row][col]<=0) 
					HH_Inc_grid[0][row][col]=0;
				else HH_Inc_grid[0][row][col]=inc_yr[yr-START_YR][row][col]/(popln[yr-START_YR][row][col]/6.0); /*	Assuming a family of 6 per HH 	 */	
			}
		}

		/* Population Calculation*/ 
		yr_tmp=yr-START_YR-1; /* START_YR-1 = 1989*/
		for(row=0; row<LINE; row++){
			for(col=0; col<PIXEL; col++){
				alpha=popln[yr_tmp][row][col]*(1.0+(gr_rate[yr_tmp]/100.0)); /*what is alpha and why 100*/
				alpha+=0.6; /* how 0.6 ????*/
				popln[yr-START_YR][row][col]=(int)(alpha);
				popln[yr-START_YR][row][col]+=In_Mig[row][col]-Out_Mig[row][col]; /*but migration()is not called ans not used. so why In_Mig and Out_Mig ???*/
			}
		}
		/* Write Population Maps */
		i_to_a(yr,tmp_str);
		strcpy(f_name,"op1/popln_"); 
		strcat(f_name,tmp_str); 
		strcat(f_name,".bil");
		printf("Finished Population Adjustments and Started writing Output to: %s\n",f_name);

		if((fplu=fopen(f_name,"wb"))==NULL){printf("Can't open %s\n",f_name); exit(0);}

		for(row=0;row<LINE;row++){
			for(col=0;col<PIXEL;col++){		/* band 1 - New LU values */ /*what does the comment mean ?*/
				print_buf[col]=popln[yr-START_YR][row][col];
			}
			fwrite(print_buf,sizeof(short),PIXEL,fplu);
		}
		fclose(fplu);

		strcpy(f_name,"op1/popln_"); 
		strcat(f_name,tmp_str); 
		strcat(f_name,".hdr");
		hdr_prn(2, 1, f_name);	/* 	dat=2; - sizeof(short); n_bands=1;*/ /* ................................FUNCTION CALL @ p2............................... */


	}	/* upto here - FOR ALL YEARS  */
} /* End of Main */
/*.....................................................................SET4 ENDS HERE .......................................................................*/
