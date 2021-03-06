# Chequerboard Travel Solution

## Solution description

The problem was attacked using Wandorf's Heuristic: the pawn is always moved to the square from which
it will have the fewest onward moves.

The solution is generic; so its easy to provide a board with any Chequerboard size
(tested with boards.size <= 20). Also it could be possible to test a solution with other Chess pieces different
from pawn: for example a Knight. For that you only need to create a figure like this:

    object Knight extends ChessPiece {
      override val getDeltas: Vector[Position] = {
        val pst = Vector((-2,-1),(-2,1),(-1,-2),(-1,2),(1,-2),(1,2),(2,-1),(2,1))
        pst.map(p => Position(p._1, p._2))
      }
    }

where getDeltas defines how the chess figure can move.

## Specs

- Scala 2.12.0
- SBT
- Scalaz

## Running the program

### Run problem instance for a Pawn with start position (2,3) on a chequerboard of size 10

    sbt "run-main com.truecaller.chequerboard.app.PawnTravelApp"

It will print something like this:

    Vector(Position(2,3), Position(0,1), Position(3,1), Position(6,1), Position(9,1),
           Position(9,4), Position(9,7), Position(7,9), Position(4,9), Position(1,9),
           Position(1,6), Position(1,3), Position(1,0), Position(4,0), Position(7,0),
           Position(9,2), Position(9,5), Position(9,8), Position(6,8), Position(3,8),
           Position(0,8), Position(0,5), Position(0,2), Position(2,0), Position(5,0),
           Position(8,0), Position(8,3), Position(8,6), Position(8,9), Position(5,9),
           Position(2,9), Position(0,7), Position(0,4), Position(2,6), Position(5,6),
           Position(5,3), Position(7,1), Position(4,1), Position(1,1), Position(1,4),
           Position(3,2), Position(6,2), Position(8,4), Position(8,1), Position(5,1),
           Position(7,3), Position(7,6), Position(5,8), Position(8,8), Position(8,5),
           Position(8,2), Position(6,4), Position(6,7), Position(3,7), Position(3,4),
           Position(1,2), Position(1,5), Position(1,8), Position(4,8), Position(7,8),
           Position(7,5), Position(7,2), Position(9,0), Position(9,3), Position(9,6),
           Position(9,9), Position(7,7), Position(7,4), Position(4,4), Position(6,6),
           Position(6,3), Position(6,0), Position(4,2), Position(4,5), Position(2,7),
           Position(0,9), Position(0,6), Position(2,8), Position(4,6), Position(2,4),
           Position(2,1), Position(4,3), Position(6,5), Position(8,7), Position(6,9),
           Position(3,9), Position(3,6), Position(5,4), Position(5,7), Position(3,5),
           Position(1,7), Position(4,7), Position(2,5), Position(2,2), Position(5,2),
           Position(5,5), Position(3,3), Position(3,0), Position(0,0), Position(0,3))

That shows the steps in order that the pawn needs to follow for solving the problem.


### How i know this solution is correct?

Because of property based testing. I find the next properties of the problem:

- Existence

Given a solution **s** with type Option[Vector[Position]] for a board of size 10 the next is an equation
to demonstrate **existence property** of the problem:
 
    - s should not be None 

- Completeness

Given a solution **s** with type Vector[Position] for a board of size **n** the next are equations 
that needs to be satisfied for demonstrate the **completeness property** of the problem:
   
    - n * n = s.size
    - s.distinct.size = s.size
    - s.toSet diff completeCoordinates = Set.empty
        where completeCoordinates are calculated as follows:
            (for (a <- 0 to s.size; b <- 0 to s.size) yield Position(a, b)).toSet
    
- Movement correctness

Given a solution **s** with type Vector[Position] for a board of size **n** and a **p** chess piece with
delta movements **d** with type Vector[Position], where delta movements codificates how the chess piece
can move; the next code demonstrate the **movement correctness property** of the problem:

      def testCorrectMovementsProperty(solution: Path, chessPiece: ChessPiece): Path = {
        def verifyStepsCorrectness(step: Position, nextStep: Position): Boolean = {
          val delta = Position(nextStep.x - step.x, nextStep.y - step.y)
          chessPiece.getDeltas contains delta
        }
        val adjacentSteps = solution.sliding(2)
        val movementsCorrectness = adjacentSteps.forall(pairPosition => verifyStepsCorrectness(pairPosition.head, pairPosition(1)))
        movementsCorrectness should be(true)
        solution
      }

The above three properties where codificated and tested in scala class:

    com.truecaller.chequerboard.domain.ChessPieceTravelSolverTest

### Run Test

    sbt test

