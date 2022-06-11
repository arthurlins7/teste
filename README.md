Implementação 2 - Designing a virtual memory manager

*Código*
<p>
  <ul>
    <li>O código tem como finalidade traduzir endereços de memória lógicos para físicos.<br />
    <li>Foi implementado na linguagem de programação C.<br />
   </ul></p>

*Código*
<p>
  <ul>
    <li>Para começar o programa foram feitas as leituras dos arquivos addresses.txt e BACKING_STORE.bin:<br />
   </ul></p>
   
```c
if ((fptr = fopen("addresses.txt","r")) == NULL){
      printf("Error! opening addresses file");
      exit(1);
}
if ((fptw = fopen("correct.txt","w")) == NULL){
      printf("Error! opening correct file");
      exit(1);
}
  
int backing_fd = open("BACKING_STORE.bin", 00);
backing = mmap(0, 256*256, 0x1, 0x02, backing_fd, 0); 
```
<p>
  <ul>
    <li>Após isso foi utilizado um while para que todas as entradas da tabela de páginas fossem = -1, ou seja, inicialização da tabela:<br />
   </ul></p>

```c
while (i < 256) {
      tabeladepaginas[i] = -1;
      i++;
  }
```
<p>
  <ul>
    <li>Então outro while foi colocado, porém esse com o intuito de que o loop funcione enquanto houver algo para ler no arquivo adresses.txt, então o número da página é retirado do endereço lógico e a tradução dos endereços virtuais e é feita a consulta da TLB: <br />
   </ul></p>

```c
while (fscanf(fptr, "%d", &enderecologico) == 1) {
        total++;
        int frame = 0, physicalad=0, hit=0, f = 0;
        int paginalogica = (enderecologico >> 8) & 255;
        int offset = enderecologico & 255;
    for(i=0; i < 16; i++)
			{
        if(tlb[i][0]==paginalogica)
			{
				hit=1;
				frame=tlb[i][1];
				break;
			}
    }
```
<p>
  <ul>
    <li>Então caso a página se encontre na TLB, é incrementado um TLB hit, caso não ela é procurada na tabela de páginas: <br />
   </ul></p>

```c
if(hit==1){ 
			tlb_hits++;
    } else {
      for(i=0; i < 256; i++){
        if(tabeladepaginas[i]==paginalogica)
				{
					frame=i;	
					break;
				}
        if(tabeladepaginas[i]==-1)
				{
					f=1;
          page_faults++;
					break;
				}
			}
			if(f==1)
			{
				tabeladepaginas[i]=paginalogica;
				frame=i;
			}
```
 <p>
  <ul>
    <li>Depois fiz a substituição por meio do FIFO: <br />
   </ul></p>

```c
tlb[s][0]=paginalogica;
			tlb[s][1]=i;
			s++;
			s=s%15;	
```
 <p>
  <ul>
    <li>A tradução dos endereços:<br />
   </ul></p>

```c
memmove(memoriaprincipal + frame * 256, backing + paginalogica * 256, 256);
      }
    
        signed char value = memoriaprincipal[frame * 256 + offset];
        enderecofisico= frame * 256 + offset;
```
 <p>
  <ul>
    <li>E por fim imprimi conforme o requisitado:<br />
   </ul></p>

```c
        fprintf(fptw, "Virtual address: %d Physical address: %d Value: %d\n", enderecologico, enderecofisico, value);
    }

    fprintf(fptw, "Number of Translated Addresses = %d\n", total);
    fprintf(fptw, "Page Faults = %d\n", page_faults);
    fprintf(fptw, "Page Fault Rate = %.3f\n", page_faults/((double)total));
    fprintf(fptw, "TLB Hits = %d\n", tlb_hits);
    fprintf(fptw, "TLB Hit Rate = %.3f\n", tlb_hits/((double)total));memmove(memoriaprincipal + frame * 256, backing + paginalogica * 256, 256);
    }
    
        signed char value = memoriaprincipal[frame * 256 + offset];
        enderecofisico= frame * 256 + offset;
```

     
*Makefile*

<p>
  <ul>
   <li>O Makefile é composto pelos comandos $make, $make run e $make clean.<br />
   <li>O comando $make compila o arquivo tipo C e gera um arquivo binário.<br />
   <li>O $make run usa o arquivo binário para executar o código.<br />
   <li>E o $make clean tem a função de apagar o arquivo binário que foi anteriormente gerado no $make.<br />
</ul></p>

```
all:
        gcc implementacao.c -o implementacao

run:
        ./implementacao

clean:
        rm correct.txt
```

*Resultado*
```
Virtual address: 16916 Physical address: 20 Value: 0
Virtual address: 62493 Physical address: 285 Value: 0
Virtual address: 30198 Physical address: 758 Value: 29
Virtual address: 53683 Physical address: 947 Value: 108
Virtual address: 40185 Physical address: 1273 Value: 0
Virtual address: 28781 Physical address: 1389 Value: 0
Virtual address: 24462 Physical address: 1678 Value: 23
Virtual address: 48399 Physical address: 1807 Value: 67
...
Number of Translated Addresses = 1000
Page Faults = 243
Page Fault Rate = 0.243
TLB Hits = 53
TLB Hit Rate = 0.053
```
