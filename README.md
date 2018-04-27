# Project
#include <iostream>
#include <string>
#include <vector>
using namespace std;
uint32_t regs[31],s1data, s2data,aluresult, memory[127];
int zeroflag=0,datamemresult;


struct BranchPredictor
{
	uint32_t pc;
	uint32_t targetaddress;
	int state;      //state 0: Stronglt NT --state 1: NT -- State 2: Taken -- State 3: Strongly Taken
};
vector <BranchPredictor> branch;

void Registersreset()                              //initialize all registers to 0
{
	for (int i = 0; i < 32; i++)
	{
		regs[i] = 0;
	}
}

void RegisterFile (uint32_t rs, uint32_t rt, uint32_t rd, uint32_t w_data, bool regwrite)    //Read and Write from registers
{
	
	if ((regwrite) && (rd<32) && (rd>=0))
	{
		regs[rd] = w_data;                                    //Write
	}
	else
		if (!(regwrite) && (rs<32) && (rt<32))
	{
			s1data = regs[rs];                             //Read
			s2data = regs[rt];
	}
}

void ALU(uint32_t operand1, uint32_t operand2, uint32_t shamt, int alucontrol)
{
	switch (alucontrol)
	{
	case 1: aluresult = operand1 + operand2; break;                 //ADD
	case 2: aluresult = operand1 - operand2; break;                 //SUB
	case 3: aluresult = operand1 & operand2; break;                 //AND
	case 4: aluresult = operand1 || operand2; break;                //OR
	case 5: aluresult = operand1 ^ operand2; break;                //XOR
	case 6: aluresult = signed(operand2) << (shamt & 0x1F); break; // SLL
	case 7: if (operand1 < operand2) { aluresult = 1; }             //SLT
			else { aluresult = 0; }break;                           
	case 8:aluresult = unsigned(operand2) >> (shamt & 0x1F); break; //SRL
	}

	if (aluresult==0)
	{
		zeroflag = 1;         //zeroflag is 1 when output is 0
	}
}

void datamemoryreset()                   //initialize all the values in the memory to 0
{
	for (int i = 0; i < 128; i++)
	{
		memory[i] = 0;
	}
}
void datamemory(uint32_t address, uint32_t data, bool memorywrite)             //Read and Write in memory, memory size is 128
{
	if ((memorywrite) && (address < 128) && (address >= 0))        //write 
	{
		memory[address] = data;
		
	}

	else
		if (!(memorywrite) && (address<128) && (address>=0))          //Read
	{
			datamemresult = memory[address];
	}
}

void checkpc(uint32_t add, uint32_t target)         //checks if pc exsits in the Branch Predictor Table 
{
	bool flag = false;
	if ((add < branch.size()) && (add >= 0))
	{
		for (int i = 0; i < branch.size(); i++)
		{
			if (branch[i].pc == add)
				flag = true;

		}
	}

	if (!(flag))            //if pc doesn't exist add it
	{
		BranchPredictor addedpc;
		addedpc.pc = add;
		addedpc.targetaddress = target;
		addedpc.state = 0;          //State is strongly Not Taken
		branch.push_back(addedpc);           //  add the pc to the branch predictor
	}


}
void updatestate(uint32_t pc, uint32_t targetadd,bool currentstate)   //updates the decision of the predictor based on the current state 
{ //if current state is 1: branch taken, else not taken
	

	for (int i = 0; i < branch.size(); i++)
	{
		if (branch[i].pc == pc)
		{
			if (currentstate)
			{
				if (branch[i].state < 3)
				{
					branch[i].state = branch[i].state + 1;
				}
			}
			else if (!(currentstate))
			{
				if (branch[i].state > 0)
				{
					branch[i].state = branch[i].state - 1;
				}
			}
		}
	}
}
int get_state(uint32_t address)      //returns the state of an address
{
	for (int i = 0; i < branch.size(); i++)
	{
		if (address == branch[i].pc)
		{
			return branch[i].state;
		}
	}
	return 0;      //not found so it will have a state of Strongly NT i.e 0
}
uint32_t get_target_address(uint32_t pc1)
{
	for (int i = 0; i < branch.size(); i++)
	{
		if (pc1 == branch[i].pc)
		{
			return branch[i].targetaddress;
		}
	}
	return pc1;  //not found so target address is the address itself
}
int main()
{
	
	system("pause");

}
