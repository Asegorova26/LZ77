import java.io.*;
import java.util.Scanner;

public class LZ77 {
    public static final int DEFAULT_BUFF_SIZE = 1024;
    protected int mBufferSize;
    protected Reader mIn;
    protected PrintWriter mOut;
    protected StringBuffer mSearchBuffer;

    public LZ77() {
        this(DEFAULT_BUFF_SIZE);
    }

    public LZ77(int buffSize) {
        mBufferSize = buffSize;
        mSearchBuffer = new StringBuffer(mBufferSize);
    }

    private void trimSearchBuffer() {
        if (mSearchBuffer.length() > mBufferSize) {
            mSearchBuffer = mSearchBuffer.delete(0,  mSearchBuffer.length() - mBufferSize);
        }
    }

    public void compress(String infile) throws IOException {
        mIn = new BufferedReader(new FileReader(infile+".txt"));
        mOut = new PrintWriter(new BufferedWriter(new FileWriter(infile+".lz77")));

        int nextChar;
        String currentMatch = "";
        int matchIndex = 0, tempIndex = 0;

        while ((nextChar = mIn.read()) != -1) {
            tempIndex = mSearchBuffer.indexOf(currentMatch + (char)nextChar);
            if (tempIndex != -1) {
                currentMatch += (char)nextChar;
                matchIndex = tempIndex;
            } else {
                String codedString =
                        "~"+matchIndex+"~"+currentMatch.length()+"~"+(char)nextChar;
                String concat = currentMatch + (char)nextChar;
                if (codedString.length() <= concat.length()) {
                    mOut.print(codedString);
                    mSearchBuffer.append(concat);
                    currentMatch = "";
                    matchIndex = 0;
                } else {
                    currentMatch = concat; matchIndex = -1;
                    while (currentMatch.length() > 1 && matchIndex == -1) {
                        mOut.print(currentMatch.charAt(0));
                        mSearchBuffer.append(currentMatch.charAt(0));
                        currentMatch = currentMatch.substring(1, currentMatch.length());
                        matchIndex = mSearchBuffer.indexOf(currentMatch);
                    }
                }
                trimSearchBuffer();
            }
        }
        if (matchIndex != -1) {
            String codedString =
                    "~"+matchIndex+"~"+currentMatch.length();
            if (codedString.length() <= currentMatch.length()) {
                mOut.print("~"+matchIndex+"~"+currentMatch.length());
            } else {
                mOut.print(currentMatch);
            }
        }
        mIn.close();
        mOut.flush(); mOut.close();
    }

    public static void main(String [] args) {
        Scanner in = new Scanner(System.in);
        System.out.println("Введите название файла:");
        String nameOfTheFile = in.next();
        LZ77 lz = new LZ77();
        try {
            lz.compress(nameOfTheFile);
        } catch (FileNotFoundException f) {
            System.err.println("File not found: ");
        } catch (IOException e) {
            System.err.println("Problem processing file: ");
        }
    }
}
